# Attach ACM Certificate to CloudFront distribution

1.	Create a ACM certificate in US-East-1 region.
	```
	aws acm request-certificate --domain-name www.example.com --validation-method DNS --region us-east-1
	```

2.	Create CNAME records where you manage your domain to complete the validation.

3. 	Run below command to check the certificate status.
	> Remove `.` from the CNAME value when copying
	```
	aws acm describe-certificate --certificate-arn <certificate-arn>
	```
4.	Attach the ACM certificate to CloudFront distribution.
	```
	aws cloudfront update-distribution \
	  --id DISTRIBUTION_ID \
	  --certificate CertificateARN \ 
	  --certificate-source ssl-management
	```
5. Go to distribution, edit, update domain name and select ACM.

4.	~Run below command to add a alternative domain name to CloudFront distribution.~
	```
	aws cloudfront associate-alias \
		--distribution-id DISTRIBUTION_ID \
		--alias ALIAS_NAME

