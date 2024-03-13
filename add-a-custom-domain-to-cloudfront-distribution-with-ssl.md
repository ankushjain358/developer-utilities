# Add a Custom Domain to CloudFront distribution with SSL


## Steps
1.	Create a ACM certificate in `us-east-1` region with below command, or via AWS Console.
	```
	aws acm request-certificate --domain-name www.example.com --validation-method DNS --region us-east-1
	```
2.	Create CNAME records where you manage your domain to complete the validation. Remove `.` from the CNAME value when copying.
	```
	aws acm describe-certificate --certificate-arn <certificate-arn> --region us-east-1
	```	
4. 	Go to CloudFront distribution, edit general settings, add **Alternate domain name (CNAME)**, select **Custom SSL certificate** and save.
5. 	Now, just add a CNAME record to your domain (for example `cdn.example.com`), and point it to the Cloudfront distribution URL.

## Clean up
1. Remove the association of ACM certification with CloudFront distribution. 
2. Delete ACM certificate from `us-east-1` region.
3. Delete CNAME record created for DNS validation from the hosted zone.
4. Delete CNAME record that point to Cloudfront distribution.

