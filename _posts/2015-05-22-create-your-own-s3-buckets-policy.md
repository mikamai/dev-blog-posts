---
ID: 120
post_title: Create your own S3 Bucket’s policy
author: Emanuel Carnevale
post_date: 2015-05-22 12:34:42
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/22/create-your-own-s3-buckets-policy/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/119597574629/create-your-own-s3-buckets-policy
tumblr_mikamayhem_id:
  - "119597574629"
---
<p>Even if you have one AWS account under which you host many websites and instances, you may want (and you should) keep your S3 buckets separated.</p>

<p>When you create a new S3 bucket it usually is accessible from any user account you created in IAM. In fact the default policy to allow access to an S3 bucket is the following (aptly named AmazonS3FullAccess):</p>

<p><code>{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}</code></p>

<p>Luckily, if we want to restrict one user to only a subset of S3 buckets, we can create a custom policy. Unfortunately it&rsquo;s not as straightforward as one would expect. Instinctively one would assume that all it&rsquo;s necessary is to substitute the globbing asterisk with the resource, like this:</p>

<p><code>{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::rails-app-production"
        },
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::rails-app-staging"
        }
    ]
}</code></p>

<p>But it doesn&rsquo;t work… We have to explicitly specify the listing policy for the buckets, so we will have:</p>

<p><code>{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::rails-app-production"
        },
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::rails-app-staging"
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "arn:aws:s3:::*"
        }
    ]
}</code></p>

<p>Using s3:ListAllMyBuckets, we say what the user can see when logs into an S3 account. If we leave the Resource value like <code>"arn:aws:s3:::*"</code> our user will be able to see all our buckets, but will access only to the ones specified before. Sometimes it can be enough, but if you don&rsquo;t want to disclose to some users what buckets you have created, you can narrow the list down.</p>

<p>Cool, right? Unfortunately it still doesn&rsquo;t work as expected, we have to complicate things a little bit more and come up with:</p>

<p><code>{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation",
                "s3:ListBucketMultipartUploads"
            ],
            "Resource": [
                "arn:aws:s3:::rails-app-production",
                "arn:aws:s3:::rails-app-staging"
            ],
            "Condition": {}
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:DeleteObject",
                "s3:DeleteObjectVersion",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:GetObjectVersion",
                "s3:GetObjectVersionAcl",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:PutObjectAclVersion"
            ],
            "Resource": [
                "arn:aws:s3:::rails-app-production/*",
                "arn:aws:s3:::rails-app-staging/*"
            ],
            "Condition": {}
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*",
            "Condition": {}
        }
    ]
}</code></p>

<p>This works great for creating a bucket for a single app which I think it should be enough for the majority of our readers and will at least avoid the hazard of having an account able to access all the buckets.
The level of detail you can control in the Amazon policies is very high and if you want to go into the rabbit hole, just follow these links:
(<a href="http://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket">http://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket</a>)[Writing IAM Policies: How to grant access to an Amazon S3 bucket]
(<a href="http://blogs.aws.amazon.com/security/post/Tx1P2T3LFXXCNB5/Writing-IAM-policies-Grant-access-to-user-specific-folders-in-an-Amazon-S3-bucke">http://blogs.aws.amazon.com/security/post/Tx1P2T3LFXXCNB5/Writing-IAM-policies-Grant-access-to-user-specific-folders-in-an-Amazon-S3-bucke</a>)[Writing IAM policies: Grant access to user-specific folders in an Amazon S3 bucket]
(<a href="http://blogs.aws.amazon.com/security/post/TxPOJBY6FE360K/IAM-policies-and-Bucket-Policies-and-ACLs-Oh-My-Controlling-Access-to-S3-Resourc">http://blogs.aws.amazon.com/security/post/TxPOJBY6FE360K/IAM-policies-and-Bucket-Policies-and-ACLs-Oh-My-Controlling-Access-to-S3-Resourc</a>)[IAM policies and Bucket Policies and ACLs! Oh My! (Controlling Access to S3 Resources]</p>