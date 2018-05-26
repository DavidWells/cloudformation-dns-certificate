# Cloudformation DNS Validated Certificate Resource

The cloudformation AWS::CertificateManager::Certificate resource can only create email validated certificates.

This is a custom cloudformation resource which can additionally create DNS validated certificates for domains that use
a Route 53 hosted zone.

## Usage

It should behave identically to AWS::CertificateManager::Certificate (see
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-certificatemanager-certificate.html).

The additional VerificationMethod property is supported which may be 'EMAIL' or 'DNS', as in the API documentation
(https://docs.aws.amazon.com/acm/latest/APIReference/API_RequestCertificate.html#ACM-RequestCertificate-request-ValidationMethod).

When using 'DNS' as the VerificationMethod the DomainValidation property becomes required. The DomainValidationOption
values no longer have a ValidationDomain but instead a HostedZoneId. The HostedZoneId should be the zone to create
the DNS validation records in.

Certificates may take up to 30 minutes to be issued, but typically takes ~3 minutes. The Certificate resource remains 
CREATING until the certificate is issued.

To use this custom resource, copy the CustomAcmCertificateLambda and CustomAcmCertificateLambdaExecutionRole resources
into your template. You can then create certificate resources of Type: AWS::CloudFormation::CustomResource using the
properties you expect. Remember to add a ServiceToken property to the resource which references the CustomAcmCertificateLambda arn.

## Examples

cloudformation.py is an example of using troposphere to create a template with a Certificate resource. 
The cloudformation.json and cloudformation.yaml files are generated from this as examples which could be used directly.
The certificate resource looks like:

    ExampleCertificate:
        Properties:
          DomainName: test.example.com
          DomainValidationOptions:
            - DomainName: test.example.com
              HostedZoneId: Z2KZ5YTUFZNC7H
          ServiceToken: !GetAtt 'CustomAcmCertificateLambda.Arn'
          Tags:
            - Key: Name
              Value: Example Certificate
          ValidationMethod: DNS
        Type: AWS::CloudFormation::CustomResource
