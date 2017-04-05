---
ID: 1320
post_title: Importing invalid legacy data with Rails
author: Andrea Longhi
post_date: 2017-04-05 09:37:02
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/04/05/importing-invalid-legacy-data-with-rails/
published: true
---
Recently I needed to import some legacy data from a list of CSV files (each file representing one database table on the legacy application)  into a fresh new application, but the legacy data was not compliant with the new application rules.

<!-- more -->

Let's consider the following example, simplified and modified for the scope of this article:
<pre><code>
class Tutor &lt; ApplicationRecord
  has_many :phones

  validates :email, presence: true

  before_save :update_default_rate

  private

  def update_default_rate
    if default_rate.present? and default_rate_changed?
      self.advanced_rate = default_rate * 1.30
    end
  end
end
</code></pre>
And the following <code>TutorImporter</code> class, which imports the legacy data, one line at a time:
<pre><code>
class TutorImporter
  def import(line)
    Tutor.create!(
      email:         line.email,
      default_rate:  line.rate
      advanced_rate: line.advanced_rate
    )
  end
end
</code></pre>
With the code above, if some validation fails, the <code>tutor</code> record will not be saved, and an exception will be raised. Of course, there may be (ahem, there should be!) constraints also at the RDBMS level, but that's out of the scope of this article.

As you may expect, the CSV data doesn't match correctly the new application constraints: for example the tutor phones information lies in another CSV file, mixed with phone numbers for other kind of users for the platform.

In order to simplify things I decided to import those phones later, but of course this decision will leave momentarily all the imported tutors without phones, making them all invalid and preventing them to be saved. The first solution that comes to mind is to change the import method as follows:
<pre><code>
class TutorImporter
  def import(line)
    tutor = Tutor.new(
      email:         line.email,
      default_rate:  line.rate,
      advanced_rate: line.advanced_rate
    )
    Tutor.save validate: false
  end
end
</code></pre>
which works like a charm: it will import all rows, with or without valid phones... but also those without emails!

When you use <code>validate: false</code> you loose completely control of what goes into the database. In my case when a tutor has no email, then that's a real issue. At the end of the day no tutor in the database should be missing the email, so those records should be discarded by the importer as well.

There is another issue there, besides validations: the <code>before_save</code> callback is not applicable when importing the legacy tutors, because they already have the <code>advanced_rate</code> value and it's different from the one calculated in the <code>update_default_rate</code> method. I don't want the callback to change the advanced rate that was originally decided for the tutor.

We now may be tempted to modify the <code>Tutor</code> class record... we may add a flag field that allows us to skip the phones validations when necessary, and the <code>before_save</code> callback as well, something like this:
<pre><code>
class Tutor &lt; ApplicationRecord
  attr_accessor :skip_phone_validations_and_callbacks

  validates :phones, presence: true, unless: -&gt; (record) { record.skip_phone_validations_and_callbacks }

  before_save :update_default_rate, unless: -&gt; (record) { record.skip_phone_validations_and_callbacks }
end
</code></pre>
This should do it. the email validation was not modified, while the other one and the callback are now optional.

There's a big drawback here: the model was changed in order to make it more flexible for a one-shot operation (the CSV import). After the import, the good developer should come back to the code and revert their changes, which is not very practical... what if the patched model files are 10? What if the changes are far more complex that the ones I showed in the example? Will the developer remember all the changes? What if somebody else has to finish their job? Will that other person know exactly what to do with those files and validations? I doubt it.

Eventually I found another solution, which I think has no major drawbacks, and has the extra benefit of communicating to the reader that there are different rules at play when importing tutors from CSV legacy data:
<pre><code>
class TutorImporter
  class Tutor &lt; ApplicationRecord
    self.table_name = 'tutors'

    validates :email, presence: true
  end

  def import(line)
    Tutor.create!(
      email:         line.email,
      default_rate:  line.rate,
      advanced_rate: line.advanced_rate
    )
  end
end
</code></pre>
Now I am importing the data using a different <code>Tutor</code> class: <code>TutorImporter::Tutor</code>. This class has only the email validation and has no before_save callback, so it will import all the CSV lines using the right criteria.

The main benefit of this solution is that CSV import logic never leaks outside of its scope, leaving the original <code>Tutor</code> class completely unchanged. You may be worried by the fact that <i>some code</i> got duplicated (the email validation in my example, but it may be much more stuff), but that can be moved in a module shared by both tutor classes.