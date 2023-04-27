# RoboPub Toolkit

## Author: complex robot
## April 25th, 2023

## Introduction

If anyone knows me they know I am a huge evangelist for AWS and the services they offer. It's hands down my favorite cloud platform, anything you can cook up can be done so easily using cloud resources. It's really an amazing time we live in, very far ahead of where things were when I first started my career.

As you also already know complex robot is all about web, web technologies, design, but also about infrastructure solutions to host a website. The problem with the S3 static website solution is that you most definitely need to be comfortable with coding your own site, uploading the assets and code, or at least working with someone else who is. This makes the barrier to entry a little high.

What if though we could bundle this up in a way that was not SIMPLE to consume (I don't think this solution will ever be *simple*), but much easier? What if there was a way we can put together a website like building blocks and get it over to our S3 bucket? If the infrastructure was already built out (ie, the S3 bucket, the Cloudfront distro) and a customer could update their own site that would be a much more compelling option, wouldn't it?

Using AWS for hosting simple websites is **incredibly** economical. And I'd go out on a limb and argue that most small to even medium sized business don't need a solution like Wix or Squarespace. Is it the most expensive way to go about getting a site on the Internet? No, but you're possibly paying for all kinds of bells and whistles you'll never, ever use.

What if I told you that in a way, with a little bit of elbow grease, you could have your cake and eat it to? What if you could build your own website and maintain it and do it on the world's leading cloud service platform with 99.99999999% uptime? And at a cost that's a *fraction* of what it would cost on a platform like Wix?

For a while now I've been thinking this would be cool, but how? I didn't really think it was possible - you need some way of building out the site. How could this be done?

Enter Mobirise.

What is Mobirise? You can read more about it here https://mobirise.com/html-builder.html. In a nutshell, Mobirise is a 'What You See Is What You Get' (WYSIWYG) website editor that can be used by anyone with a keen eye and a desire to maintain their own website. The beautiful thing here is that you can edit a website offline - no Internet connection is needed. It works on both Windows and Mac (sorry Linux folks!), and most importantly, it can *perform a full export of assets and pages for your site* into any directory you want. This is the secret sauce. Combining a local system export with a small set of connector tools I've developed is the key to getting your site from your machine into AWS' hosting platform.

![Easy as ABC?](./artifacts/robopub001.jpg)

**Is it as easy as ABC?** Unfortunately, no. This is still going to take a little determination on your end. You're going to be doing a little bit of command line surfing. You're going to be running a script. I am hopeful that future versions will be even better. But you will learn something! And learning is good! Plus, you will be able to manage your own website without paying anyone!

The intent of this document is to record my progress in the design of this tool. I'll be sharing it with anyone/everyone regardless. Once it's up and running I intend on either doing videos or a nice document on the operation and setup of the toolkit. It won't just be this. 

Thank you for taking this journey with me :) buckle up! We've got a bit to go over!





===---___---===---___---===

This is a pretty simple tutorial to get you up and running serving a static website out of an S3 bucket on Amazon Web Services (AWS)! You will need a couple of things to accomplish this:

1. Be sure you have an AWS account. It could be a production account, test, dev, whatever you please. Best to not implement anything in a production account until you're comfortable with the process, but any AWS account will do.
2. You WILL need a registered domain name and the DNS zone MUST be hosted on Route 53. You could register the DNS elsewhere and then host your zone on AWS but that's outside the scope of this roblog entry. This guide assumes you have registered a domain name on AWS Route 53 and also have the DNS hosted zone configured there as well.

The purpose of this guide is to walk through the setup of a static website using S3 website endpoints in Amazon Web Services. "Static" is a bit of a misnomer in this context, for many people static means non-interactive, no movement, etc, but this is not the case with a web site. Static in this case simply means there's no dynamic allocation of assets, no server side computation, no ability to interface with a back end (natively). However, anything that can run client side (JavaScript, HTML and CSS primarily) *can* interact with web elements and the DOM which will allow for a very rich, interactive user experience.  

## How-To
As above, please *please* make sure you have already registered a domain name and will be hosting the DNS on Route 53. This is absolutely imperative since Route 53 will be calling the resource in a way that will only work from within AWS hosted DNS (rather than the typical IP reference setup it will use an Alias A record which is AWS specific for pointing to AWS resources and services).
* Log into the AWS console for the account you will be setting this up on. For the purposes of this guide I will be using my personal AWS account with a portfolio website I'd like to set up *(iammane.link)*
* Click in to Route 53 to ensure your domain and hosted zone are configured as such (crucially, you want to ensure there is a "NS" type DNS record and "SOA" (these SHOULD be populated by default just by nature of registering a domain and setting up a hosted zone on Route 53).

![Route 53 initial settings](./artifacts/s3_site_01.jpg)
* Move over to the S3 console and create two buckets for your site. The bucket names should match the domain you are working with (again, in this example, *iammane.link*). You will want to create two buckets - one will be the same name as your domain, the other will be www.yourdomain.com (this is used later for a redirect). **Note: All defaults for other settings are ok as is for now. We'll make adjustments to the bucket policy later, but for now just get these buckets made**.

![iammane.link bucket](./artifacts/s3_site_02.jpg)
* At this point your bucket configuration should look like the below

![iammane.link buckets](./artifacts/s3_site_03.jpg)
* Click into your **NON** www bucket (ie, for me this is *iammane.link*). Click on upload and put all of your static site content into the bucket. Be sure to include all files and folders! Once you've selected everything click on Upload. Navigate back to the main bucket root and click on the Properties link at the top.

![Click on Properties from within bucket root](./artifacts/s3_site_04.jpg)
* Scroll down to the very bottom of the properties page and click Edit for "Static website hosting". Click the radio button to enable static website hosting and verify the "Host a static website" radio button is selected (should be by default). Enter the name for your index document (this is the main root HTML file for your website that you should have uploaded earlier) and put in an error document optionally if you so desire. Click "Save changes" at the bottom of this screen. See below:

![Static hosting setup](./artifacts/s3_site_05.jpg)
* Click back into buckets and click on the "www" variant of your website. Click again on the Properties tab at the top, scroll down to the "Static website hosting section again and click on Edit. You will once again select Enable for "Static website hosting", but this time you will choose the radio button for "Redirect requests for an object". Enter the non www variant of your bucket where you uploaded all the content and select "Save changes". The protocol selection doesn't make a difference for this architecture. Your screen should look at below before hitting save.

![Set up the redirect](./artifacts/s3_site_06.jpg)
* Next we will modify the bucket policy for each of these (the main site content bucket and the redirect bucket). Currently neither of these buckets are open for public access, which for a hosted website is pretty useless! In order to make these available for public viewing we will need to add a bucket policy onto each. Click on the Permissions tab at the top for the main content bucket. Scroll down to the "Block public access (bucket settings)" section and click Edit.

![Make the bucket public](./artifacts/s3_site_07.jpg)
* Remove the checkmark from "Block *all* public access". Make sure none of the other options are checked and click "Save changes". 

![Confirm public bucket settings](./artifacts/s3_site_08.jpg)
* You will be presented with the above. AWS is very careful to make sure you understand that allowing a bucket to be public means that **when you configure it** external entities and services will be able to access this bucket. This does not immediately make your bucket public on the wide open Internet, it simply means that the bucket may now be used in a public access fashion.
* Scroll down to the "Bucket policy" section and click Edit. Paste in the following to be used as the bucket policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::Bucket-Name/*"
            ]
        }
    ]
}
```

The above will allow ALL principals to have s3:GetObject rights over the bucket you name. This is required to allow all entities on the Internet to view your site. The screen should resemble the below but with **your** bucket name in the Resource section of the bucket policy:

![iammane.link with bucket policy](./artifacts/s3_site_09.jpg)
* Click on "Save changes" at the bottom. 
* Head back to your S3 buckets and note that your main site content bucket now reads Public with a warning symbol. This is due to the fact we applied a bucket policy that allows anonymous read (again, required to read the files from your site).

![iammane.link and www.iammane.link difference](./artifacts/s3_site_10.jpg)
* **PERFORM THE SAME STEPS FOR YOUR WWW BUCKET! This includes turning off the "Block public access (bucket settings)" check mark and the addition of the bucket policy to your www.yourdomain.com bucket!**
* Once both of your buckets have been configured with the bucket policy to allow anonymous read access we need to make our final changes to Route 53. Click into the Route 53 console and open up the hosted zone for your domain name (in my case, *iammane.link*).
* We will need to create two A records for our buckets, they are going to point to the S3 website endpoints we configured earlier. Start by clicking on "Create record"

![iammane.link hosted zone create record](./artifacts/s3_site_11.jpg)
* If you are dropped into the Wizard view click the link that will read "Switch to quick create".
* Leave the *subdomain* value blank, select record type of A, click the toggle for Alias and choose the "S3 website endpoint" value under "Route traffic to" drop down box.
* Choose the region your S3 bucket is in (for me this is us-east-1).
* Click into the "*Enter S3 endpoint*" box and it should auto populate with the website endpoint that matches the non www bucket (again, in my case *iammane.link*) - see below, yours should match outside of the domain name in use. 

![Create the record for iammane.link](./artifacts/s3_site_12.jpg)
* "Routing policy" should stay as the default "Simple routing" and flip the toggle to off for "Evaluate target health". Click on "Create records"
* You'll be brought back to the hosted zone for your domain. Simply click on "Create record" again and perform the same exact steps but this time the *subdomain* entry will be www for the redirect. See below for the example of what iammane.link looks like:

![Complete DNS for the redirect bucket](./artifacts/s3_site_13.jpg)

As long as DNS for your domain name has propagated through the public Internet the changes we have made about should happen very quickly, typically within a matter of 60 seconds. Browse to your site using either *yourdomain.com* or *www.yourdomain.com* and the site should appear!

![The completed site](./artifacts/s3_site_14.jpg)

## Recap

This was a lot of steps, but let's quickly summarize what we accomplished:

1. Created a main website content bucket (yourdomain.com) and a www redirect bucket (www.yourdomain.com). This is required because www.yourdomain.com would not resolve to your website otherwise. This subdomain could actually be whatever you want, but most sites follow the convention of yourdomain.com or www.yourdomain.com for access.
2. Uploaded all of our site content into the yourdomain.com bucket.
3. Set up static web hosting on both buckets, using the www one as a redirect to your main content bucket.
4. Disabled the block public access setting on both buckets.
5. Configured both buckets with an open-ended, anonymous public access read policy.
6. Configured Route 53 DNS A records to point to the alias S3 website endpoints for the buckets.

## Important notes and limitations

A keen eye will immediately notice that the site will not run in HTTPS mode. Websites hosted out of S3 cannot **natively** run as HTTPS, but there is a workaround for this using another AWS service that I'll detail in another guide.

Another issue is that since the bucket is configured for open, anonymous internet access there is nothing stopping an automated process from scraping the same file over and over and over out of your S3 bucket. From a security and cost perspective this is very less than ideal since you will pay for object requests out of an S3 bucket to the public internet. The same mechanism for getting your site to run as an HTTPS endpoint also addresses this issue, so in a way it's a two for one special 😃 in production you will want to use this method, but to quickly stand up a site in short order this guide is perfect.

One other caveat is that S3 can *only* be used to serve static assets. What this means is that using an S3 bucket as a website will only allow you to program a site to run client side code, serve up HTML and CSS. There is no mechanism for back channel communication, no server/client model, none of the trimmings you would get from a more robust architecture. That said though, in most cases a static website would serve more than 90% of the small to medium sized business that are looking to embrace a cost effective technology to either establish or migrate their web presence.

Thanks for reading!

February 13th, 2023