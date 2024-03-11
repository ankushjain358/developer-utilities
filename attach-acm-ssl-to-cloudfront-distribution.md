# Attach ACM Certificate to CloudFront distribution

1.	Create a ACM certificate in `us-east-1` region with below command, or via AWS Console.
	```
	aws acm request-certificate --domain-name www.example.com --validation-method DNS --region us-east-1
	```
2.	Create CNAME records where you manage your domain to complete the validation. Remove `.` from the CNAME value when copying.
	```
	aws acm describe-certificate --certificate-arn <certificate-arn>
	```	
4. 	Go to CloudFront distribution, edit general settings, add custom domain name and select ACM certificate, and save.

## Clean up
1. Remove the association of ACM certification with CloudFront distribution. 
1. Delete ACM certificate from `us-east-1` region.
2. Delete CNAME records created for DNS validation from the hosted zone

