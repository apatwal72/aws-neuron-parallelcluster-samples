# Create ParallelCluster

Once your prerequisite infrastructure and AMI are all set up, you are ready to create a ParallelCluster. Copy the following content into a launch.yaml file in your local desktop where AWS ParallelCluster CLI is installed:

```
Region: <YOUR REGION> # i.e., us-west-2
Image:
  Os: alinux2
  CustomAmi: ami-<AML2 PARALLELCLUSTER AMI ID>
SharedStorage:
  - Name: myebs
    StorageType: Ebs
    MountDir: /shared
    EbsSettings:
      VolumeType: gp2
      Size: 20
HeadNode:
  InstanceType: c5.2xlarge
  Networking:
    SubnetId: subnet-<PUBLIC SUBNET ID>
  Ssh:
    KeyName: <KEY NAME WIHTOUT .PEM>
  LocalStorage:
    RootVolume:
      Size: 1024
Scheduling:
  Scheduler: slurm
  SlurmQueues:
    - Name: compute1
      CapacityType: ONDEMAND
      ComputeSettings:
        LocalStorage:
          RootVolume:
            Size: 1024
          EphemeralVolume:
            MountDir: /local_storage
      ComputeResources:
        - Efa:
            Enabled: true
          InstanceType: trn1.32xlarge
          MaxCount: 16
          MinCount: 1
          Name: queue1-i1
      Networking:
        SubnetIds:
          - subnet-<PRIVATE SUBNET ID>
        PlacementGroup:
          Enabled: false
SharedStorage:
- EfsSettings:
    ProvisionedThroughput: 1024
    ThroughputMode: provisioned
  MountDir: /efs
  Name: ktfefs
  StorageType: Efs
  ```

The yaml file above will create a ParallelCluster with one c5.2xlarge head node, and 16 trn1.32xl compute nodes. All 16 trn1 nodes are in the same queue. In case you need to isolate compute nodes with different queues, simply append another instanceType designation to the current instanceType, and designate `MaxCount` for each queue, for example, `InstanceType` section would be become:

```
InstanceType: trn1.32xlarge
MaxCount: 8
MinCount: 0
Name: queue-0
InstanceType: trn1.32xlarge
MaxCount: 8
MinCount: 0
Name: queue-1
```

So now you have two queues, each queue is designated to a number of trn1 compute nodes. An unique feature for trn1.32xlarge instance is the EFA interfaces built for high performance/low latency network data transfer. This is indicated by:

```
- Efa:
    Enabled: true
```

If you are using trn1.2xl instance, this feature is not enabled, and in which case, you don’t need such designation.

You also need to designate an EC2 private key . This is indicated by the following line in launch.yaml :

```
Ssh:
  KeyName: trn1
```

In the virtual environment where you installed AWS ParallelCluster API, run the following command:

```
pcluster create-cluster --cluster-configuration launch.yaml \
--cluster-name My-ParallelCluster-Trn1 \
--suppress-validators type:ComputeResourceLaunchTemplateValidator \
\
```
Where

`cluster-configuration` is the path to yaml file

`cluster-name` is the name of your cluster

`supress-validators` is used here to generalize this command so it will not run into error triggered by tagging policies, if any.

This will create a ParallelCluster in your AWS account, and you may inspect the progress in AWS CloudFormation console.

Please follow the sections ["Setting up the training environment on trn1.32xlarge"](https://awsdocs-neuron-staging.readthedocs-hosted.com/en/release_2.3.0rc2/frameworks/torch/tutorials/training/bert.html?next=https%3A%2F%2Fawsdocs-neuron-staging.readthedocs-hosted.com%2Fen%2Frelease_2.3.0rc1%2Fframeworks%2Ftorch%2Ftutorials%2Ftraining%2Fbert.html%3Fnext%3Dhttps%253A%252F%252Fawsdocs-neuron-staging.readthedocs-hosted.com%252Fen%252Frelease_2.3.0rc1%252Fframeworks%252Ftorch%252Ftutorials%252Ftraining%252Fbert.html&ticket=ST-1663365027-jWyjPKGS3TtpDY9Ih0iklXykKnHRSSnL#id4) and ["Downloading tokenized and sharded dataset files"](https://awsdocs-neuron-staging.readthedocs-hosted.com/en/release_2.3.0rc2/frameworks/torch/tutorials/training/bert.html?next=https%3A%2F%2Fawsdocs-neuron-staging.readthedocs-hosted.com%2Fen%2Frelease_2.3.0rc1%2Fframeworks%2Ftorch%2Ftutorials%2Ftraining%2Fbert.html%3Fnext%3Dhttps%253A%252F%252Fawsdocs-neuron-staging.readthedocs-hosted.com%252Fen%252Frelease_2.3.0rc1%252Fframeworks%252Ftorch%252Ftutorials%252Ftraining%252Fbert.html&ticket=ST-1663365027-jWyjPKGS3TtpDY9Ih0iklXykKnHRSSnL#id5) to setup the BERT scripts and download the dataset files.