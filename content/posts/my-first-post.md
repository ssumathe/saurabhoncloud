+++
date = '2025-05-19T22:23:46+05:30'
draft = true
title = 'EKS node Encrypted volumes in AWS'
+++
Recently, I was working on an assignment to fix the encryption issue noticed in one of the security scans.   
AWS was creating the EKS nodes with unecrypted volume type and we learned that with eksctl commands to create the eks nodes AWS by default creates unencrypted volume as they recommend clients or users to manage the encryption (which i think should be by defualt encrypted.). 

Now to fix this in our operational lower environments we had 2 options which I want to describe in this blog:  

1. Using Custom launch template to update the volume type to encrypted. And use this launch template to create the nodeGroup in eks.  

To create a launch template using json you can use the below json
```
aws ec2 create-launch-template \
  --launch-template-name TemplateForEncryption \
  --launch-template-data file://config.json
```
  and json can be given as below, note that Encrypted: true, you can use this template to even update volumeType from gp2 to gp3.  

```
  {
    "BlockDeviceMappings":[
        {
            "DeviceName":"/dev/sda1",
            "Ebs":{
                "VolumeType":"gp2",
                "DeleteOnTermination":true,
                "SnapshotId":"snap-066877671789bd71b",
                "Encrypted":true,
                "KmsKeyId":"arn:aws:kms:us-east-1:012345678910:key/abcd1234-a123-456a-a12b-a123b4cd56ef"
            }
        }
    ],
    "ImageId":"ami-00068cd7555f543d5",
    "InstanceType":"c5.large",
    "TagSpecifications":[
        {
            "ResourceType":"volume",
            "Tags":[
                {
                    "Key":"encrypted",
                    "Value":"yes"
                }
            ]
        }
    ]
}
```
2. Another way is to enable EBS encryption on account level so that any volume created is Encrypted by default. It includes both EBS volumes for ec2, nodes and also EBS PVCs if any.  
To enable it you can go to ec2 dashboard --> data-protection and security --> EBS encryption. (this is regional setting).  
For EFS PVCs in EKS, this can be done during the EFS File system creation in AWS.
