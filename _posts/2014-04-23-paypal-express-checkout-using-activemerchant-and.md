---
ID: 256
post_title: >
  Paypal express checkout using
  ActiveMerchant and Instant Payment
  Notification (IPN)
author: Matteo Zobbi
post_date: 2014-04-23 14:38:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/23/paypal-express-checkout-using-activemerchant-and/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/83620379034/paypal-express-checkout-using-activemerchant-and
tumblr_mikamayhem_id:
  - "83620379034"
---
Instant Payment Notification (IPN) is a message service that notifies you of events related to PayPal transactions.<!--more-->

You can use IPN messages to automate back-office and administrative functions, such as fulfilling orders, tracking customers, and providing status and other transaction-related information or other kind of things, like emailing your customers upon purchase and offers.

Remember that if you are using Express Checkout or Direct Payment, the PayPal API notifies you of the status and details of the transaction immediately and automatically. This means that you need not use IPN for knowing the transaction status, because you will know the transaction status immediately.

This is the IPN flow:

<img src="http://68.media.tumblr.com/e20234d51c42d604e70f2637798f3ede/tumblr_inline_n4hh66iDKR1r5gu9j.gif" alt="image" />

1 A user clicks a PayPal button to kick off a checkout flow; your web application makes an API call; your back-office system makes an API call; or PayPal observes an event.

2 PayPal HTTP POSTs your listener a message that notifies you of this event.

3 Your listener returns an empty HTTP 200 response.

4 Your listener HTTP POSTs the complete, unaltered notification back to the PayPal.

<em>Note: This message must contain the same fields, in the same order, as the original notification, all preceded by cmd=_notify-validate. Further, this message must use the same encoding as the original.</em>

5 PayPal sends a single word back - either VERIFIED (if the message matches the original) or INVALID (if the message does not match the original).

IPN sends post data to the IPN listener so… have a route defined for this.

This is a basic payment method:
<pre><code>
def payment
  item = Item.find(params[:id])
  response = EXPRESS_GATEWAY.setup_purchase(item.price_in_cents,{
    :ip                =&gt; request.remote_ip,
    :currency_code     =&gt; '...',
    :return_url        =&gt; success_url(item)
    :notify_url        =&gt; notify_url(item),
    :cancel_return_url =&gt; cancel_url(item)
    [...]
    })
  item.update_attribute :express_token, response.token
  redirect_to EXPRESS_GATEWAY.redirect_url_for(response.token)
end
</code></pre>
This is the success url:
<pre><code>
def success
  details = EXPRESS_GATEWAY.details_for(express_token)
  response = EXPRESS_GATEWAY.purchase(item.price_in_cents,{
    :ip               =&gt; request.remote_ip,
    :currency_code    =&gt; '...',
    :token            =&gt; express_token,
    :payer_id         =&gt; details.payer_id
    […]
  })

  if response.success?
    [...]
    flash[:success] = "Your transaction was successfully completed"
    [...]
  else
    flash[:error] = "Your transaction could not be compelted"
    [...]
  end

end
</code></pre>
…and here is a typical handler for IPN with ActiveMerchant:
<pre><code>
class BackendController &lt; ApplicationController
  include ActiveMerchant::Billing::Integrations

  def paypal_ipn
    notify = Paypal::Notification.new(request.raw_post)

    order = Order.find(notify.item_id)

    if notify.acknowledge
      begin

        if notify.complete? and order.total == notify.amount
          order.status = 'success'

          shop.ship(order)
        else
          logger.error("Failed to verify Paypal's notification, please investigate")
        end

      rescue =&gt; e
        order.status        = 'failed'
        raise
      ensure
        order.save
      end
    end

    render :nothing
  end
end

</code></pre>
One more thing: aknowledge method has to be called after a new ipn arrives. Paypal will verify that all the information we received are correct and will return a ok or a fail.

This is the acknowledge method on ActiveMerchant module:
<pre><code>
def acknowledge(authcode = nil)
  payload =  raw

  response = ssl_post(Paypal.service_url + '?cmd=_notify-validate', payload,
    'Content-Length' =&gt; "#{payload.size}",
    'User-Agent'     =&gt; "Active Merchant -- <a href="http://activemerchant.org">http://activemerchant.org</a>"
  )

  raise StandardError.new("Faulty paypal result: #{response}") unless ["VERIFIED", "INVALID"].include?(response)

  response == "VERIFIED"
end
</code></pre>
Here are some references

ActiveMerchant Notification module: <a href="https://github.com/Shopify/active_merchant/blob/master/lib/active_merchant/billing/integrations/paypal/notification.rb">https://github.com/Shopify/active_merchant/blob/master/lib/active_merchant/billing/integrations/paypal/notification.rb</a>

ActiveMerchant IPN Documentation: <a href="http://activemerchant.rubyforge.org/classes/ActiveMerchant/Billing/Integrations/Paypal/Notification.html">http://activemerchant.rubyforge.org/classes/ActiveMerchant/Billing/Integrations/Paypal/Notification.html</a>

Paypal IPN Documentation: <a href="https://developer.paypal.com/docs/classic/ipn/gs_IPN/">https://developer.paypal.com/docs/classic/ipn/gs_IPN/</a>