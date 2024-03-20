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

more to come later. Perhaps.