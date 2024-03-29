```conf
# Unique name of Kubernetes cluster. In order to deploy
# more than one cluster into the same AWS account, this
# name must not conflict with an existing cluster.
clusterName: egov-micro-dev

# CoreOS release channel to use. Currently supported options: alpha, beta, stable
# See coreos.com/releases for more information
#releaseChannel: stable

# The AMI ID of CoreOS.
# If omitted, the latest AMI for the releaseChannel is used.
#amiId: ""
# The ID of hosted zone to add the externalDNSName to.
# Either specify hostedZoneId or hostedZone, but not both
#hostedZoneId: ""

# Network ranges of sources you'd like SSH accesses to be allowed from, in CIDR notation. Defaults to ["0.0.0.0/0"] which allows any sources.
# Explicitly set to an empty array to completely disable it.
# If you do that, probably you would like to set securityGroupIds to provide this worker node pool an existing SG with a SSH access allowed from specific ranges.
#sshAccessAllowedSourceCIDRs:
#- 0.0.0.0/0

# The name of one of API endpoints defined in `apiEndpoints` below to be written in kubeconfig and then used by admins
# to access k8s API from their laptops, CI servers, or etc.
# Required if there are 2 or more API endpoints defined in `apiEndpoints`
#adminAPIEndpointName: versionedPublic

# Kubernetes API endpoints with each one has a DNS name and is with/without a managed/unmanaged ELB, Route 53 record set
# CAUTION: `externalDNSName` must be omitted when there are one or more items under `apiEndpoints`
apiEndpoints:
- # The unique name of this API endpoint used to identify it inside CloudFormation stacks or
  # to be referenced from other parts of cluster.yaml
  name: default

  # DNS name for this endpoint, added to the kube-apiserver TLS cert
  # It must be somehow routable to the Kubernetes controller nodes
  # from worker nodes and external clients. Configure the options
  # below if you'd like kube-aws to create a Route53 record sets/hosted zones
  # for you.  Otherwise the deployer is responsible for making this name routable
  dnsName: k8s-dev.egovernments.org

  # Configuration for an ELB serving this endpoint
  # Omit all the settings when you want kube-aws not to provision an ELB for you
  loadBalancer:
    # Specifies an existing load-balancer used for load-balancing controller nodes and serving this endpoint
    # Setting id requires all the other settings excluding `name` to be omitted because reusing an ELB implies that configuring other resources
    # like a Route 53 record set for the endpoint is now your responsibility!
    # Also, don't forget to add controller.securityGroupIds to include a glue SG to allow your existing ELB to access controller nodes created by kube-aws
    #id: existing-elb

    # Set to true when you want kube-aws to create a Route53 ALIAS record set for this API load balancer for you
    # Must be omitted when `id` is specified
    createRecordSet: false
    # All the subnets assigned to this load-balancer. Specified only when this load balancer is not reused but managed one
    # Must be omitted when `id` is specified
    #subnets:
    #- name: managedPublic1

    # Set to true so that the managed ELB becomes an `internal` one rather than `internet-facing` one
    # When set to true while subnets are omitted, one or more private subnets in the top-level `subnets` must exist
    # Must be omitted when `id` is specified
    #private: false

    # TTL in seconds for the Route53 RecordSet created if createRecordSet is set to true.
    #recordSetTTL: 300

    # The Route 53 hosted zone is where the resulting Alias record is created for this endpoint
    # Must be omitted when `id` is specified
    #hostedZone:
    #  # The ID of hosted zone to add the dnsName to.
    #  id: ""
#    # Network ranges of sources you'd like Kubernetes API accesses to be allowed from, in CIDR notation. Defaults to ["0.0.0.0/0"] which allows any sources.
#    # Explicitly set to an empty array to completely disable it.
#    # If you do that, probably you would like to set securityGroupIds to provide this load balancer an existing SG with a Kubernetes API access allowed from specific ranges.
#    apiAccessAllowedSourceCIDRs:
#    #- 0.0.0.0/0
#
#    # Existing security groups attached to this load balancer which are typically used to
#    # allow Kubernetes API accesses from admins and/or CD systems when `apiAccessAllowedSourceCIDRs` are explicitly set to an empty array
#    securityGroupIds:
#    - sg-1234567
#
#  #
#  # Common configuration #1: Unversioned, internet-facing API endpoint
#  #
#  - name: unversionedPublic
#    dnsName: api.example.com
#    loadBalancer:
#      id: elb-abcdefg
#
#  #
#  # Common configuration #2: Versioned, internet-facing API endpoint
#  #
#  - name: versionedPublic
#    dnsName: v1api.example.com
#    loadBalancer:
#      hostedZone:
#        id: hostedzone-abcedfg
#
#  #
#  # Common configuration #3: Unmanaged endpoint a.k.a Extra DNS name added to the kube-apiserver TLS cert
#  #
#  - name: extra
#    dnsName: youralias.example.com

# Name of the SSH keypair already loaded into the AWS
# account being used to deploy this cluster.
keyName: egov-k8s-sample

# Additional keys to preload on the coreos account (keep this to a minimum)
# sshAuthorizedKeys:
# - "ssh-rsa AAAAEXAMPLEKEYEXAMPLEKEYEXAMPLEKEYEXAMPLEKEYEXAMPLEKEYEXAMPLEKEYEXAMPLEKEYEXAMPLEKEYEXAMPLEKEYEXAMPLEKEY example@example.org"

# Region to provision Kubernetes cluster
region: ap-southeast-1

# Availability Zone to provision Kubernetes cluster when placing nodes in a single availability zone (not highly-available) Comment out for multi availability zone setting and use the below `subnets` section instead.
availabilityZone: ap-southeast-1b

# ARN of the KMS key used to encrypt TLS assets.
kmsKeyArn: "arn:aws:kms:ap-southeast-1:xxaccount-idxx:key/xxxxxx-my-key-xxxxxxx"

#controller:
#  # Number of controller nodes to create, for more control use `controller.autoScalingGroup` and do not use this setting
#  count: 1
#
#  # Maximum time to wait for controller creation
#  createTimeout: PT15M
#
#  # Instance type for controller node.
#  # CAUTION: Don't use t2.micro or the cluster won't work. See https://github.com/kubernetes/kubernetes/issues/18975
#  instanceType: t2.medium
#
#  rootVolume:
#    # Disk size (GiB) for controller node
#    size: 30
#    # Disk type for controller node (one of standard, io1, or gp2)
#    type: gp2
#    # Number of I/O operations per second (IOPS) that the controller node disk supports. Leave blank if controller.rootVolume.type is not io1
#    iops: 0
#
#  # Existing security groups attached to controller nodes which are typically used to
#  # (1) allow access from controller nodes to services running on an existing infrastructure
#  # (2) allow ssh accesses from bastions when `sshAccessAllowedSourceCIDRs` are explicitly set to an empty array
#  securityGroupIds:
#    - sg-1234abcd
#    - sg-5678efab
#
#  # Auto Scaling Group definition for controllers. If only `controllerCount` is specified, min and max will be the set to that value and `rollingUpdateMinInstancesInService` will be one less.
#  autoScalingGroup:
#    minSize: 1
#    maxSize: 3
#    rollingUpdateMinInstancesInService: 2
#  # If you specify managedIamRoleName the role created for controller nodes will not suffix the random id at the end
#  # Role will be created with "Ref": {"AWS::StackName"}-{"AWS::Region"}-yourManagedRole
#  # to follow the recommendation in AWS documentation  http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html
#  # This is also intended to be used in combination with .experimental.kube2IamSupport. See #297 for more information.
#  #
#  # ATTENTION: Consider limiting number of characters in clusterName and iam.role.name to avoid the resulting IAM
#  # role name's length from exceeding the AWS limit: 64
#  # See https://github.com/kubernetes-incubator/kube-aws/issues/347
#  # if you omit this block kube-aws would create an IAM Role and a customer managed policy with a random name.
#  #iam:
#    # role:
#       # This will create a named IAM Role managed by kube-aws with the name {AWS::Region}-$managedIamRoleName.
#       # It will have attached a customer Managed Policy that you can modify afterwards if you need more permissions for your cluster.
#       # Be careful with the Statements you modify because an update could overwrite your own.
#       # the Statements included in the ManagedPolicy are the minimun ones required for the Controllers to run.
#       # name: "yourManagedRole"
#       # if you set managedPolicies here it will be attached in addition to the created managedPolicy in kube-aws for the cluster.
#       # CAUTION: if you attach a more restrictive policy in some resources (i.e ec2:* Deny) you can make kube-aws fail.
#       # managedPolicies:
#       # -  arn: "arn:aws:iam::aws:policy/AdministratorAccess"
#       # -  arn: "arn:aws:iam::YOURACCOUNTID:policy/YOURPOLICYNAME"

#       # if you set an InstanceProfile kube-aws will NOT create any IAM Role and will use the configured instanceProfile.
#       # CAUTION: you must ensure that the IAM Role linked to the listed InstanceProfile has enough permissions to ensure kube-aws to run.
#       # if you dont know which permissions are required is recommended to create a cluster with a managed role.
#       # instanceProfile:
#       #   arn: "arn:aws:iam::YOURACCOUNTID:instance-profile/INSTANCEPROFILENAME"

#  # If omitted, public subnets are created by kube-aws and used for controller nodes
#  subnets:
#    # References subnets defined under the top-level `subnets` key by their names
#    - name: ManagedPublicSubnet1
#    - name: ManagedPublicSubnet2
#
#   # Kubernetes node labels to be added to controller nodes
#   nodeLabels:
#     kube-aws.coreos.com/role: controller
#
#  # User defined files that will be added to the Controller cluster cloud-init configuration in the "write_files:" section.
#  customFiles:
#    - path: "/etc/rkt/auth.d/docker.json"
#      permissions: 0600
#      content: |
#        { "rktKind": "dockerAuth", "rktVersion": "v1", ... }
#
#  # User defined systemd units that will be added to the Controller cluster cloud-init configuration in the "units:" section.
#  customSystemdUnits:
#    - name: monitoring.service
#      command: start
#      enable: true
#      content: |
#        [Unit]
#        Description=Example Custom Service
#        [Service]
#        ExecStart=/bin/rkt run --set-env TAGS=Controller ...

# Default number of worker nodes per node pool to create, for more control use `worker.nodePools[].count` and/or `worker.autoScalingGroup` and do not use this setting
#workerCount: 1

worker:
#
#  # Settings that apply to all worker node pools
#
#  # The name of the API endpoint defined in the top level `apiEndpoints` that the worker node pools will use to access the k8s API
#  # Required if there are 2 or more API endpoints defined in `apiEndpoints`
#  # Can be overriden per node pool by specifying `worker.nodePools[].apiEndpointName`
#  apiEndpointName: versionedPublic
#
  nodePools:
    - # Name of this node pool. Must be unique among all the node pools in this cluster
      name: nodepool1
#      # If omitted, public subnets are created by kube-aws and used for worker nodes
#      subnets:
#      - # References subnets defined under the top-level `subnets` key by their names
#        name: ManagedPublicSubnet1
#
#      # Existing "glue" security groups attached to worker nodes which are typically used to allow
#      # access from worker nodes to services running on an existing infrastructure
#      securityGroupIds:
#        - sg-1234abcd
#        - sg-5678efab
#
#      # Configuration for external managed ELBs for worker nodes
#      # Use this with k8s load balancers with type=NodePort. See https://kubernetes.io/docs/user-guide/services/#type-nodeport
#      #
#      # NOTICE: This is generally recommended over k8s managed load-balancers with type=LoadBalancer because it allows
#      # a manual but no-downtime rotation of kube-aws clusters
#      loadBalancer:
#        enabled: true
#        # Names of ELBs attached to worker nodes
#        names: [ "manuallymanagedelb" ]
#        # IDs of "glue" security groups attached to worker nodes to allow the ELBs to communicate with worker nodes
#        securityGroupIds: [ "sg-87654321" ]
#
#      # The name of the API endpoint defined in the top level `apiEndpoints` that this worker node pool will use to access the k8s API
#      # If not specified, defaults to `worker.apiEndpointName`
#      apiEndpointName: versionedPublic
#
#     # if you omit this block kube-aws would create an IAM Role and a customer managed policy with a random name.
#      #iam:
#       # role:
#       # This will create a named IAM Role managed by kube-aws with the name {AWS::Region}-$managedIamRoleName.
#       # It will have attached a customer Managed Policy that you can modify afterwards if you need more permissions for your cluster.
#       # Be careful with the Statements you modify because an update could overwrite your own.
#       # the Statements included in the ManagedPolicy are the minimun ones required for the Controllers to run.
#       #   name: "yourManagedRole"
#       # if you set managedPolicies here it will be attached in addition to the created managedPolicy in kube-aws for the cluster.
#       # CAUTION: if you attach a more restrictive policy in some resources (i.e ec2:* Deny) you can make kube-aws fail.
#       #   managedPolicies:
#       #     - arn: "arn:aws:iam::aws:policy/AdministratorAccess"
#       #     - arn: "arn:aws:iam::YOURACCOUNTID:policy/YOURPOLICYNAME"

#       # if you set an InstanceProfile kube-aws will NOT create any IAM Role and will use the configured instanceProfile.
#       # CAUTION: you must ensure that the IAM Role linked to the listed InstanceProfile has enough permissions to ensure kube-aws to run.
#       # if you dont know which permissions are required is recommended to create a cluster with a managed role.
#       # instanceProfile:
#       #   arn: "arn:aws:iam::YOURACCOUNTID:instance-profile/INSTANCEPROFILENAME"
#      # Configuration for external managed ALBs Target Groups for worker nodes
#      targetGroup:
#        enabled: true
#        # ARNs of ALBs Target Groups attached to worker nodes
#        arns:
#        - arn:aws:elasticloadbalancing:eu-west-1:xxxxxxxxxxxx:targetgroup/manuallymanagedetg/xxxxxxxxxxxxxxxx
#        # IDs of "glue" security groups attached to worker nodes to allow the ALBs Target Groups to communicate with worker nodes
#        securityGroupIds: [ "sg-87654321" ]
#
#      # If you specify managedIamRoleName the role created for worker nodes will not suffix the random id at the end
#      # Role will be created with "Ref": {"AWS::StackName"}-{"AWS::Region"}-yourManagedRole
#      # to follow the recommendation in AWS documentation  http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html
#      # This is also intended to be used in combination with kube2IamSupport. See #297 for more information.
#      #
#      # ATTENTION: Consider limiting number of characters in clusterName, nodePools[].name and managedIamRoleName to
#      # avoid the resulting IAM role name's length from exceeding the AWS limit: 64
#      # See https://github.com/kubernetes-incubator/kube-aws/issues/347
#      managedIamRoleName: "yourManagedRole"
#
#      # Additional EBS volumes mounted on the worker
#      # No additional EBS volumes by default. All parameter values do not default - they must be explicitly defined
#      volumeMounts:
#      - type: "gp2"
#        iops: 0
#        size: 30
#        # Follow the aws convention of '/dev/xvd*' where '*' is a single letter from 'f' to 'z'
#        device: "/dev/xvdf"
#        path: "/ebs"
#
#      # Specifies how often kubelet posts node status to master. Note: be cautious when changing the constant, it must work with nodeMonitorGracePeriod in nodecontroller.
#      # nodeStatusUpdateFrequency: "10s"
#
#      #
#      # Settings only for ASG-based node pools
#      #
#
#      # Number of worker nodes to create for an autoscaling group based pool
#      count: 1
#
#      # Instance type for worker nodes
#      # CAUTION: Don't use t2.micro or the cluster won't work. See https://github.com/kubernetes/kubernetes/issues/16122
      instanceType: m4.large
#
#      rootVolume:
#        # Disk size (GiB) for worker nodes
#        size: 30
#        # Disk type for worker node (one of standard, io1, or gp2)
#        type: gp2
#        # Number of I/O operations per second (IOPS) that the worker node disk supports. Leave blank if worker.rootVolume.type is not io1
#        iops: 0
#
#      # Maximum time to wait for worker creation
#      createTimeout: PT15M
#
#      # Tenancy of the worker nodes. Options are "default" and "dedicated"
#      # Documentation: http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/dedicated-instance.html
#      tenancy: default
#
#      # (Experimental) GPU Driver installation support
#      # Currently, only Nvidia driver is supported.
#      # This setting takes effect only when configured instance type is GPU enabled (p2 or g2).
#      # Make sure to choose 'docker' as container runtime when enabled this feature.
#      # Ensure that automatic Container Linux is disabled(it is disabled by default btw).
#      # Otherwise the installed driver may stop working when an OS update resulted in an updated kernel
#      gpu:
#        nvidia:
#          enabled: true
#          version: "375.66"
#
#      # Price (Dollars) to bid for spot instances. Omit for on-demand instances.
#      spotPrice: "0.05"
#
#      # When enabled, autoscaling groups managing worker nodes wait for nodes to be up and running.
#      # This is enabled by default.
#      waitSignal:
#        enabled: true
#        # Max number of nodes concurrently updated
#        maxBatchSize: 1
#
#      # Auto Scaling Group definition for workers. If only `workerCount` is specified, min and max will be the set to that value and `rollingUpdateMinInstancesInService` will be one less.
      autoScalingGroup:
        minSize: 1
        maxSize: 4
        rollingUpdateMinInstancesInService: 3
#
#      #
#      # Spot fleet config for worker nodes
#      #
#      spotFleet:
#        # Total desired number of units to maintain
#        # An unit is chosen by you and can be a vCPU, specific amount of memory, size of instance store, etc., according to your requirement.
#        # Omit or put zero to disable the spot fleet support and use an autoscaling group to manage nodes.
#        targetCapacity: 10
#
#        # IAM role to grant the Spot fleet permission to bid on, launch, and terminate instances on your behalf
#        # See http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet-requests.html#spot-fleet-prerequisites
#        #
#        # Defaults to "arn:aws:iam::youraccountid:role/aws-ec2-spot-fleet-role" assuming you've arrived "Spot Requests" in EC2 Dashboard
#        # hence the role is automatically created for you
#        iamFleetRoleArn: "arn:aws:iam::youraccountid:role/kube-aws-doesnt-create-this-for-you"
#
#        # Price per unit hour = Price per instance hour / num of weighted capacity for the instance
#        # Defaults to "0.06"
#        spotPrice: 0.06
#
#        # Disk size (GiB) per unit
#        # Disk size for each launch specification defaults to unitRootVolumeSize * weightedCapacity
#        unitRootVolumeSize: 30
#
#        # Disk type for worker node (one of standard, io1, or gp2)
#        # Can be overridden in each launch specification
#        rootVolumeType: gp2
#
#        # Number of I/O operations per second (IOPS) per unit. Leave blank if rootVolumeType is not io1
#        # IOPS for each launch specification defaults to unitRootVolumeIOPS * weightedCapacity
#        unitRootVolumeIOPS: 0
#
#        launchSpecifications:
#        - # Number of units provided by an EC2 instance of the instanceType.
#          weightedCapacity: 1
#          instanceType: c4.large
#
#          # Price per instance hour
#          # Defaults to worker.spotFleet.spotPrice * weightedCapacity if omitted
#          #spotPrice:
#
#          rootVolume:
#            # Defaults to worker.spotFleet.rootVolumeType if omitted
#            type:
#            # Defaults to worker.spotFleet.unitRootVolumeSize * weightedCapacity if omitted
#            size:
#            # Number of provisioned IOPS when rootVlumeType is io1
#            # Must be within the range between 100 and 2000
#            # Defaults to worker.spotFleet.unitRootVolumeIOPS * weightedCapacity if omitted
#            iops:
#
#        - weightedCapacity: 2
#          instanceType: c4.xlarge
#
#      #
#      # Optional settings for both ASG-based and SpotFleet-based node pools
#      #
#
#      # Autoscaling by adding/removing nodes according to resource usage
#      autoscaling:
#        # Make this node pool an autoscaling-target of k8s cluster-autoscaler
#        #
#        # Beware that making this an autoscaing-target doesn't automatically deploy cluster-autoscaler itself -
#        # turn on `addons.clusterAutoscaler.enabled` to deploy it on controller nodes.
#        clusterAutoscaler:
#          enabled: true
#
#      # Used to provide `/etc/environment` env vars with values from arbitrary CloudFormation refs
#      awsEnvironment:
#        enabled: true
#        environment:
#          CFNSTACK: '{ "Ref" : "AWS::StackId" }'
#
#      # Add predefined set of labels to the nodes
#      # The set includes names of launch configurations and autoscaling groups
#      awsNodeLabels:
#        enabled: true
#
#      # Will provision worker nodes with IAM permissions to run cluster-autoscaler and add node labels so that
#      # cluster-autoscaler pods are scheduled to nodes in this node pool
#      clusterAutoscalerSupport:
#        enabled: true
#
#      # Mount an EFS only to a specific node pool.
#      # This is a NFS share that will be available across a node pool through a hostPath volume on the "/efs" mountpoint
#      #
#      # If you'd like to configure an EFS mounted to every node, use the top-level `elasticFileSystemId` instead
#      #
#      # Beware that currently it doesn't work for a node pool in subnets managed by kube-aws.
#      # In other words, this node pool must be in existing subnets managed by you by specifying subnets[].id to make this work.
#      elasticFileSystemId: fs-47a2c22e
#
#      # This option has not yet been tested with rkt as container runtime
#      ephemeralImageStorage:
#        enabled: true
#
#      # When enabled it will grant sts:assumeRole permission to the IAM roles for worker nodes.
#      # This is intended to be used in combination with managedIamRoleName. See #297 for more information.
#      kube2IamSupport:
#        enabled: true
#
#      # Propagate custom CLI options to kubelet
#      # Provided example overrides default docker image garbage collection limits
#      # as in https://kubernetes.io/docs/concepts/cluster-administration/kubelet-garbage-collection/
#      # Anything from https://kubernetes.io/docs/admin/kubelet/ can be used as a value
#
#      kubeletOpts: --image-gc-low-threshold=50 --image-gc-high-threshold=65
#
#      # Kubernetes node labels to be added to worker nodes
#      nodeLabels:
#        kube-aws.coreos.com/role: worker
#
#      # Kubernetes node taints to be added to worker nodes
#      taints:
#        - key: dedicated
#          value: search
#          effect: NoSchedule
#
#      # Other less common customizations per node pool
#      # All these settings default to the top-level ones
#      keyName:
#      releaseChannel: alpha
#      amiId:
#      kubernetesVersion: 1.6.0-alpha.1
#
#      # Images are taken from controlplane by default, but you can override values for node pools here. E.g.:
#      AwsCliImage:
#        repo: quay.io/coreos/awscli
#        tag: edge
#        rktPullDocker: false
#
#      sshAuthorizedKeys:
#      # User-provided YAML map available in control-plane's stack-template.json
#      customSettings:
#        key1: [ 1, 2, 3 ]

#      # User defined files that will be added to the NodePool cloud-init configuration in the "write_files:" section.
#      customFiles:
#        - path: "/etc/rkt/auth.d/docker.json"
#          permissions: 0600
#          content: |
#            { "rktKind": "dockerAuth", "rktVersion": "v1", ... }
#
#      # User defined systemd units that will be added to the NodePool cluster cloud-init configuration in the "units:" section.
#      customSystemdUnits:
#        - name: monitoring.service
#          command: start
#          enable: true
#          content: |
#            [Unit]
#            Description=Example Custom Service
#            [Service]
#            ExecStart=/bin/rkt run --set-env TAGS=Controller ...

# Maximum time to wait for worker creation
#workerCreateTimeout: PT15M

# Instance type for worker nodes
# CAUTION: Don't use t2.micro or the cluster won't work. See https://github.com/kubernetes/kubernetes/issues/16122
#workerInstanceType: t2.medium

# Disk size (GiB) for worker nodes
#workerRootVolumeSize: 30

# Disk type for worker node (one of standard, io1, or gp2)
#workerRootVolumeType: gp2

# Number of I/O operations per second (IOPS) that the worker node disk supports. Leave blank if workerRootVolumeType is not io1
#workerRootVolumeIOPS: 0

# Tenancy of the worker nodes. Options are "default" and "dedicated"
# Documentation: http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/dedicated-instance.html
# workerTenancy: default

# Price (Dollars) to bid for spot instances. Omit for on-demand instances.
# workerSpotPrice: "0.05"

## Etcd Cluster config
## WARNING: Any changes to etcd parameters after the cluster is first created will not be applied
## during a cluster upgrade, due to concerns over data loss.
## This situation is being rectified with work towards automated management of etcd clusters

etcd:
#  # Number of etcd nodes
#  # (Set to an odd number >= 3 for HA control plane)
#  # WARNING: You can't add etcd nodes after the first creation by modifying this and running `kube-aws update`
#  # See https://github.com/kubernetes-incubator/kube-aws/issues/631 for more information
  count: 3
#
#  # Instance type for etcd node
#  instanceType: t2.medium
#
#  rootVolume:
#    # Root volume size (GiB) for etcd node
#    size: 30
#    # Root volume type for etcd node (one of standard, io1, or gp2)
#    type: gp2
#    # Number of I/O operations per second (IOPS) that the etcd node's root volume supports. Leave blank if etcdRootVolumeType is not io1
#    iops: 0
#
#  dataVolume:
#    # Data volume size (GiB) for etcd node
#    # if etcdDataVolumeEphemeral=true, this value is ignored. The size of ephemeral volumes is not configurable.
#    size: 30
#    # Data volume type for etcd node (one of standard, io1, or gp2)
#    type: gp2
#    # Number of I/O operations per second (IOPS) that the etcd node's data volume supports. Leave blank if etcdDataVolumeType is not io1
#    iops: 0
#    # Encrypt the Etcd Data volume.  Set to true for encryption.  This does not work for etcdDataVolumeEphemeral.
#    encrypted: false
#    # Use ephemeral instance storage for etcd data volume instead of EBS?
#    # (Recommended set to true for high-throughput control planes)
#    # CAUTION: Currently broken. Please don't turn this on until it is fixed in https://github.com/kubernetes-incubator/kube-aws/pull/417
#    ephemeral: false
#
#  # Tenancy of the etcd instances. Options are "default" and "dedicated"
#  # Documentation: http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/dedicated-instance.html
#  tenancy: default
#
#  # If omitted, public subnets are created by kube-aws and used for etcd nodes
#  subnets:
#    # References subnets defined under the top-level `subnets` key by their names
#    - name: ManagedPrivateSubnet1
#    - name: ManagedPrivateSubnet2
#
#  # Existing security groups attached to etcd nodes which are typically used to
#  # allow ssh accesses from bastions when `sshAccessAllowedSourceCIDRs` are explicitly set to an empty array
#  securityGroupIds:
#    - sg-1234abcd
#    - sg-5678efab
#
#  # If you omit this block kube-aws would create an IAM Role and a managed policy for etcd nodes with a random name.
#  iam:
#    role:
#      # If you set managedPolicies here they will be attached to the role in addition to the managed policy.
#      # CAUTION: if you attach a more restrictive policy in some resources (i.e ec2:* Deny) you can make kube-aws fail.
#      managedPolicies:
#      - arn: "arn:aws:iam::aws:policy/AdministratorAccess"
#      - arn: "arn:aws:iam::YOURACCOUNTID:policy/YOURPOLICYNAME"
#    # If you set an InstanceProfile kube-aws will NOT create any IAM Role and will use the existing instance profile.
#    # CAUTION: you must ensure that the IAM role linked to the existing instance profile has enough permissions to ensure etcd nodes to run.
#    # If you dont know which permissions are required, it is recommended to go ahead with a managed instance profile.
#    instanceProfile:
#      arn: "arn:aws:iam::YOURACCOUNTID:instance-profile/INSTANCEPROFILENAME"
#
#  # The version of etcd to be used. Set to e.g. "3.2.1" to use etcd3
#  version: 3.2.6
#
#  snapshot:
#    # Set to true to periodically take an etcd snapshot
#    # Beware that this can be enabled only for etcd 3+
#    # Please carefully test if it works as you've expected when being enabled for your production clusters
#    automated: false
#
#  disasterRecovery:
#    # Set to true to automatically execute a disaster-recovery process whenever etcd node(s) seemed to be broken for a while
#    # Beware that this can be enabled only for etcd 3+
#    # Please carefully test if it works as you've expected when being enabled for your production clusters
#    automated: false
#
#  # The strategy to provide your etcd nodes, in combination with floating EBS volumes, stable member identities. Defaults to "eip".
#  #
#  # Available options: eip, eni
#  #
#  # With every option, etcd nodes communicate to each other via their private IPs.
#  #
#  # eip: Use EC2 public hostnames stabilized with EIPs and "resolved eventually to private IPs"
#  #      Requires Amazon DNS (at the second IP of your VPC, e.g. 10.0.0.2) to work.
#  #      1st recommendation because less moving parts and relatively easy disaster recovery for a single-AZ etcd cluster.
#  #      If you run a single-AZ etcd cluster and the AZ failed, EBS volumes created from latest snapshots and EIPs can be reused in an another AZ to reproduce your etcd cluster in the AZ.
#  #
#  # eni: [EXPERIMENTAL] Use secondary ENIs and Route53 record sets to provide etcd nodes stable hostnames
#  #      Requires Amazon DNS (at the second IP of your VPC, e.g. 10.0.0.2) or an another DNS which can resolve dns names in the hosted zone managed by kube-aws to work.
#  #      2nd recommendation because relatively easy disaster recovery for a single-AZ etcd cluster but more moving parts than "eip".
#  #      If you run a single-AZ etcd cluster and the AZ failed, EBS volumes created from latest snapshots and record sets can be reused in an another AZ to reproduce your etcd cluster in the AZ.
#  #
#  memberIdentityProvider: eip
#
#  # Domain of the hostname used for etcd peer discovery.
#  # Used only when `memberIdentityProvider: eni` for TLS key/cert generation
#  # If omitted, defaults to "ec2.internal" for us-east-1 region and "<region>.compute.internal" otherwise
#  internalDomainName:
#
#  # Set to `false` to disable creation of record sets.
#  # Used only when `memberIdentityProvider` is set to `eni`
#  # When disabled, it's your responsibility to configure all the etcd nodes so that
#  # they can resolve each other's FQDN(specified via the below `etcd.nods[].fqdn` settings) via your DNS(can be the Amazon DNS or your own DNS. Configure it with e.g. coroes-cloudinit)
#  manageRecordSets:
#
#  # Advanced configuration used only when `memberIdentityProvider: eni`
#  hostedZone:
#    # The hosted zone where record sets for etcd nodes managed by kube-aws are created
#    # If omitted, kube-aws creates a hosted zone for you
#    id:
#
#  # CAUTION: Advanced configuration. This should be omitted unless you have very deep knowledge of etcd and kube-aws
#  nodes:
#  - # The name of this etcd node. Specified only when you want to customize the etcd member's name shown in ETCD_INITIAL_CLUSTER and ETCD_NAME
#    name: etcd0
#    # The FQDN of this etcd node
#    # Usually this should be omitted so that kube-aws can choose a proper value.
#    # Specified only when `memberIdentityProvider: eni` and `manageRecordSets: false` i.e.
#    # it is your responsibility to properly configure EC2 instances to use a DNS which is able to resolve the FQDN.
#    fqdn: etcd0.<internalDomainName>
#  - name: etcd1
#    fqdn: etcd1.<internalDomainName>
#  - name: etcd2
#    fqdn: etcd2.<internalDomainName>
#
#  # ARN of the KMS key used to encrypt the etcd data volume. The account default key will be used if `etcdDataVolumeEncrypted`
#  # is enable and `etcd.kmsKeyArn` is omitted.
#  kmsKeyArn: ""
#
#  # User defined files that will be added to the Etcd cluster cloud-init configuration in the "write_files:" section.
#  customFiles:
#    - path: "/etc/rkt/auth.d/docker.json"
#      permissions: 0600
#      content: |
#        { "rktKind": "dockerAuth", "rktVersion": "v1", ... }
#
#  # User defined systemd units that will be added to the Etcd cluster cloud-init configuration in the "units:" section.
#  customSystemdUnits:
#    - name: monitoring.service
#      command: start
#      enable: true
#      content: |
#        [Unit]
#        Description=Example Custom Service
#        [Service]
#        ExecStart=/bin/rkt run --set-env TAGS=Controller ...

## Networking config

# CAUTION: Deprecated and will be removed in v0.9.9. Please use vpc.id instead
# vpcId:

#vpc:
#  # ID of existing VPC to create subnet in. Leave blank to create a new VPC
#  id:
#  # Exported output's name from another stack
#  # Only specify either id or idFromStackOutput but not both
#  #idFromStackOutput: myinfra-Vpc

# CAUTION: Deprecated and will be removed in v0.9.9. Please use internetGateway.id instead
# internetGatewayId:

#internetGateway:
#  # ID of existing Internet Gateway to associate subnet with. Leave blank to create a new Internet Gateway
#  id:
#  # Exported output's name from another stack
#  # Only specify either id or idFromStackOutput but not both
#  #idFromStackOutput: myinfra-igw

# Advanced: ID of existing route table in existing VPC to attach subnet to.
# Leave blank to use the VPC's main route table.
# This should be specified if and only if vpcId is specified.
#
# IMPORTANT NOTICE:
#
# If routeTableId is specified, it's your responsibility to add an appropriate route to
# an internet gateway(IGW) or a NAT gateway(NGW) to the route table.
#
# More concretely,
# * If you like to make all the subnets private, pre-configure an NGW yourself and add a route to the NGW beforehand
# * If you like to make all the subnets public, pre-configure an IGW yourself and add a route to the IGW beforehand
# * If you like to mix private and public subnets, omit routeTableId but specify subnets[].routeTable.id per subnet
#
# routeTableId: rtb-xxxxxxxx

# CIDR for Kubernetes VPC. If vpcId is specified, must match the CIDR of existing vpc.
# vpcCIDR: "10.0.0.0/16"

# CIDR for Kubernetes subnet when placing nodes in a single availability zone (not highly-available) Leave commented out for multi availability zone setting and use the below `subnets` section instead.
# instanceCIDR: "10.0.0.0/24"

# Kubernetes subnets with their CIDRs and availability zones.
# Differentiating availability zone for 2 or more subnets result in high-availability (failures of a single availability zone won't result in immediate downtimes)
# subnets:
#   #
#   # Managed public subnet managed by kube-aws
#   #
#   - name: ManagedPublicSubnet1
#     # Set to false if this subnet is public
#     # private: false
#     availabilityZone: us-west-1a
#     instanceCIDR: "10.0.0.0/24"
#
#   #
#   # Managed private subnet managed by kube-aws
#   #
#   - name: ManagedPrivateSubnet1
#     # Set to true if this subnet is private
#     private: true
#     availabilityZone: us-west-1a
#     instanceCIDR: "10.0.1.0/24"
#
#   #
#   # Advanced: Unmanaged/existing public subnet reused but not managed by kube-aws
#   #
#   # An internet gateway(igw) and a route table contains the route to the igw must have been properly configured by YOU.
#   # kube-aws tries to reuse the subnet specified by id or idFromStackOutput but kube-aws never modify the subnet
#   #
#   - name: ExistingPublicSubnet1
#     # Beware that `availabilityZone` can't be omitted; it must be the one in which the subnet exists.
#     availabilityZone: us-west-1a
#     # ID of existing subnet to be reused.
#     # availabilityZone should still be provided but instanceCIDR can be omitted when id is specified.
#     id: "subnet-xxxxxxxx"
#     # Exported output's name from another stack
#     # Only specify either id or idFromStackOutput but not both
#     #idFromStackOutput: myinfra-PublicSubnet1
#
#   #
#   # Advanced: Unmanaged/existing private subnet reused but not managed by kube-aws
#   #
#   # A nat gateway(ngw) and a route table contains the route to the ngw must have been properly configured by YOU.
#   # kube-aws tries to reuse the subnet specified by id or idFromStackOutput but kube-aws never modify the subnet
#   #
#   - name: ExistingPrivateSubnet1
#     # Beware that `availabilityZone` can't be omitted; it must be the one in which the subnet exists.
#     availabilityZone: us-west-1a
#     # Existing subnet.
#     id: "subnet-xxxxxxxx"
#     # Exported output's name from another stack
#     # Only specify either id or idFromStackOutput but not both
#     #idFromStackOutput: myinfra-PrivateSubnet1
#
#   #
#   # Advanced: Managed private subnet with an existing NAT gateway
#   #
#   # kube-aws tries to reuse the ngw specified by id or idFromStackOutput
#   # by adding a route to the ngw to a route table managed by kube-aws
#   #
#   # Please be sure that the NGW is properly deployed. kube-aws will never modify ngw itself.
#   #
#   - name: ManagedPrivateSubnetWithExistingNGW
#     private: true
#     availabilityZone: us-west-1a
#     instanceCIDR: "10.0.1.0/24"
#     natGateway:
#       id: "ngw-xxxxxxxx"
#       # Exported output's name from another stack
#       # Only specify either id or idFromStackOutput but not both
#       #idFromStackOutput: myinfra-PrivateSubnet1
#
#   #
#   # Advanced: Managed private subnet with an existing NAT gateway
#   #
#   # kube-aws tries to reuse the ngw specified by id or idFromStackOutput
#   # by adding a route to the ngw to a route table managed by kube-aws
#   #
#   # Please be sure that the NGW is properly deployed. kube-aws will never modify ngw itself.
#   # For example, kube-aws won't assign a pre-allocated EIP to the existing ngw for you.
#   #
#   - name: ManagedPrivateSubnetWithExistingNGW
#     private: true
#     availabilityZone: us-west-1a
#     instanceCIDR: "10.0.1.0/24"
#     natGateway:
#       # Pre-allocated NAT Gateway. Used with private subnets.
#       id: "ngw-xxxxxxxx"
#       # Exported output's name from another stack
#       # Only specify either id or idFromStackOutput but not both
#       #idFromStackOutput: myinfra-PrivateSubnet1
#
#   #
#   # Advanced: Managed private subnet with an existing EIP for kube-aws managed NGW
#   #
#   # kube-aws tries to reuse the EIP specified by eipAllocationId
#   # by associating the EIP to a NGW managed by kube-aws.
#   # Please be sure that kube-aws won't assign an EIP to an existing NGW i.e.
#   # either natGateway.id or eipAllocationId can be specified but not both.
#   #
#   - name: ManagedPrivateSubnetWithManagedNGWWithExistingEIP
#     private: true
#     availabilityZone: us-west-1a
#     instanceCIDR: "10.0.1.0/24"
#     natGateway:
#       # Pre-allocated EIP for NAT Gateways. Used with private subnets.
#       eipAllocationId: eipalloc-xxxxxxxx
#
#   #
#   # Advanced: Managed private subnet with an existing route table
#   #
#   # kube-aws tries to reuse the route table specified by id or idFromStackOutput
#   # by assigning this subnet to the route table.
#   #
#   # Please be sure that it's your responsibility to:
#   # * Configure an AWS managed NAT or a NAT instance or an another NAT and
#   # * Add a route to the NAT to the route table being reused
#   #
#   # i.e. kube-aws neither modify route table nor create other related resources like
#   # ngw, route to nat gateway, eip for ngw, etc.
#   #
#   - name: ManagedPrivateSubnetWithExistingRouteTable
#     private: true
#     availabilityZone: us-west-1a
#     instanceCIDR: "10.0.1.0/24"
#     routeTable:
#       # Pre-allocated route table
#       id: "rtb-xxxxxxxx"
#       # Exported output's name from another stack
#       # Only specify either id or idFromStackOutput but not both
#       #idFromStackOutput: myinfra-PrivateRouteTable1
#
#   #
#   # Advanced: Managed public subnet with an existing route table
#   #
#   # kube-aws tries to reuse the route table specified by id or idFromStackOutput
#   # by assigning this subnet to the route table.
#   #
#   # Please be sure that it's your responsibility to:
#   # * Configure an internet gateway(IGW) and
#   # * Attach the IGW to the VPC you're deploying to
#   # * Add a route to the IGW to the route table being reused
#   #
#   # i.e. kube-aws neither modify route table nor create other related resources like
#   # igw, route to igw, igw vpc attachment, etc.
#   #
#   - name: ManagedPublicSubnetWithExistingRouteTable
#     availabilityZone: us-west-1a
#     instanceCIDR: "10.0.1.0/24"
#     routeTable:
#       # Pre-allocated route table
#       id: "rtb-xxxxxxxx"
#       # Exported output's name from another stack
#       # Only specify either id or idFromStackOutput but not both
#       #idFromStackOutput: myinfra-PublicRouteTable1


# CIDR for all service IP addresses
# serviceCIDR: "10.3.0.0/24"

# CIDR for all pod IP addresses
# podCIDR: "10.2.0.0/16"

# IP address of Kubernetes dns service (must be contained by serviceCIDR)
# dnsServiceIP: 10.3.0.10

# Uncomment to provision nodes without a public IP. This assumes your VPC route table is setup to route to the internet via a NAT gateway.
# If you did not set vpcId and routeTableId the cluster will not bootstrap.
# mapPublicIPs: false

# Expiration in days from creation time of TLS assets. By default, the CA will
# expire in 10 years and the server and client certificates will expire in 1
# year.
#tlsCADurationDays: 3650
#tlsCertDurationDays: 365

# Use custom images for kube-aws  and  kubernetes  components. Especially if you are deploying in cn-north-1 where gcr.io is blocked
# and pulling from quay or dockerhub is slow and you get many timeouts.

# Version of hyperkube image to use. This is the tag for the hyperkube image repository.
# kubernetesVersion: v1.7.4_coreos.0

# Hyperkube image repository to use.
# hyperkubeImage:
#   repo: quay.io/coreos/hyperkube
#   rktPullDocker: false

# AWS CLI image repository to use.
# awsCliImage:
#   repo: quay.io/coreos/awscli
#   tag: master
#   rktPullDocker: false

# Calico Node image repository to use.
#calicoNodeImage:
#  repo: quay.io/calico/node
#  tag: v1.2.0
#  rktPullDocker: false

# Calico CNI image repository to use.
#calicoCniImage:
#  repo: quay.io/calico/cni
#  tag: v1.8.3
#  rktPullDocker: false

# Calico Policy Controller image repository to use.
#calicoPolicyControllerImage:
#  repo: quay.io/calico/kube-policy-controller
#  tag: v0.6.0
#  rktPullDocker: false

# Cluster Autoscaler image repository to use.
#clusterAutoscalerImage:
#  repo: gcr.io/google_containers/cluster-autoscaler
#  tag: v0.6.0
#  rktPullDocker: false

# Cluster Proportional Autoscaler image repository to use.
#clusterProportionalAutoscalerImage:
#  repo: gcr.io/google_containers/cluster-proportional-autoscaler-amd64
#  tag: 1.1.1
#  rktPullDocker: false

# kube2iam image repository to use.
#kube2iamImage:
#  repo: jtblin/kube2iam
#  tag: 0.7.0
#  rktPullDocker: false

# kube DNS image repository to use.
#kubeDnsImage:
#  repo: gcr.io/google_containers/kubedns-amd64
#  tag: 1.14.4
#  rktPullDocker: false

# kube DNS Masq image repository to use.
#kubeDnsMasqImage:
#  repo: gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64
#  tag: 1.14.4
#  rktPullDocker: false

# kube rescheduler image repository to use.
#kubeReschedulerImage:
#  repo: gcr.io/google-containers/rescheduler
#  tag: v0.3.1
#  rktPullDocker: false

# DNS Masq metrics image repository to use.
#dnsMasqMetricsImage:
#  repo: gcr.io/google_containers/k8s-dns-sidecar-amd64
#  tag: 1.14.4
#  rktPullDocker: false

# Exec Healthz image repository to use.
#execHealthzImage:
#  repo: gcr.io/google_containers/exechealthz-amd64
#  tag: 1.2
#  rktPullDocker: false

# Helm image repository to use.
#helmImage:
#  repo: quay.io/kube-aws/helm
#  tag: v2.6.0
#  rktPullDocker: false

# Tiller (Helm backend) image repository to use.
#tillerImage:
#  repo: gcr.io/kubernetes-helm/tiller
#  tag: v2.6.0
#  rktPullDocker: false

# Heapster image repository to use.
#heapsterImage:
#  repo: gcr.io/google_containers/heapster
#  tag: v1.4.1
#  rktPullDocker: false

# Addon Resizer image repository to use.
#addonResizerImage:
#  repo: gcr.io/google_containers/addon-resizer
#  tag: 2.0
#  rktPullDocker: false

# Kube Dashboard image repository to use.
#kubeDashboardImage:
#  repo: gcr.io/google_containers/kubernetes-dashboard-amd64
#  tag: v1.6.3
#  rktPullDocker: false

# Calico Controller image repository to use.
#calicoCtlImage:
#  repo: calico/ctl
#  tag: v1.2.0
#  rktPullDocker: false

# Pause image repository to use.This works only if you are deploying your cluster in "cn-north-1" region.
#pauseImage:
#  repo: gcr.io/google_containers/pause-amd64
#  tag: 3.0
#  rktPullDocker: false

# Flannel image repository to use.This allow pulling flannel image from a docker repo.
#flannelImage:
#  repo: quay.io/coreos/flannel
#  tag: v0.7.1
#  rktPullDocker: false

# JournaldCloudWatchLogsImage image repository to use. This sends journald logs to CloudWatch.
#journaldCloudWatchLogsImage:
#  repo: "jollinshead/journald-cloudwatch-logs"
#  tag: "0.1"
#  rktPullDocker: true

# Use Calico for network policy.
# useCalico: false

# Create MountTargets to subnets managed by kube-aws for a pre-existing Elastic File System (Amazon EFS),
# and then mount to every node.
#
# This is for mounting an EFS to every node.
# If you'd like to mount an EFS only to a specific node pool, set `worker.nodePools[].elasticFileSystemId` instead.
#
# Enter the resource id, eg "fs-47a2c22e"
# This is a NFS share that will be available across the entire cluster through a hostPath volume on the "/efs" mountpoint
#
# You can create a new EFS volume using the CLI:
# $ aws efs create-file-system --creation-token $(uuidgen)
#
# Beware that:
# * an EFS file system can not be mounted from multiple subnets from the same availability zone and
#   therefore this feature won't work like you might have expected when you're deploying your cluster to an existing VPC and
#   wanted to mount an existing EFS to the subnet(s) created by kube-aws
#   See https://github.com/kubernetes-incubator/kube-aws/issues/208 for more information
# * kube-aws doesn't create MountTargets for existing subnets(=NOT managed by kube-aws). If you'd like to bring your own subnets
#   to be used by kube-aws, it is now your responsibility to configure the subnets, including creating MountTargets, accordingly
#   to your requirements
#elasticFileSystemId: fs-47a2c22e

# Create shared persistent volume
#sharedPersistentVolume: false

# Determines the container runtime for kubernetes to use. Accepts 'docker' or 'rkt'.
# containerRuntime: docker

# If you do not want kube-aws to manage certificaes, set it to false. If you do that
# you are responsible for making sure that nodes have correct certificates by the time
# daemons start up.
# manageCertificates: true

# When enabled, autoscaling groups managing controller nodes wait for nodes to be up and running.
# It is enabled by default.
#waitSignal:
#  enabled: true
#   maxBatchSize: 1

# Autosaves all Kubernetes resources (in .json format) to a bucket 's3:.../<your-cluster-name>/backup/*'.
# The autosave process executes on start-up and repeats every 24 hours.
#kubeResourcesAutosave:
#  enabled: false

# When enabled, all nodes will forward journald logs to AWS CloudWatch.
# It is disabled by default.
#cloudWatchLogging:
# enabled: false
# retentionInDays: 7
# # When enabled, feedback from Journald logs (with an applied filter) will be outputted during kube-aws 'apply | up'.
# # It is enabled by default - provided cloudWatchLogging is enabled.
# localStreaming:
#  enabled: true
#  filter:  `{ $.priority = "CRIT" || $.priority = "WARNING" && $.transport = "journal" && $.systemdUnit = "init.scope" }`
#  interval: 60

# When enabled, all nodes will have Amazon SSM Agent installed.
# It is disabled by default.
# You might want to set 'managedPolicies' up to run the agents properly. More details: http://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-configuring-access-policies.html
#amazonSsmAgent:
#  enabled: true
#  downloadUrl: https://github.com/DailyHotel/amazon-ssm-agent/releases/download/v2.0.805.1/ssm.linux-amd64.tar.gz
#  sha1sum: a6fff8f9839d5905934e7e3df3123b54020a1f5e

# When enabled, will enable a DNS-masq DaemonSet to make PODs to resolve DNS names via locally running dnsmasq
# It is disabled by default.
#kubeDns:
# nodeLocalResolver: false

# When enabled, CloudFormation events will stream to stdout during kube-aws 'update | up'.
# It is enabled by default.
#cloudFormationStreaming: true

# Addon features
addons:
  # Will provision controller nodes with IAM permissions to run cluster-autoscaler and
  # create a cluster-autoscaler deployment in the cluster
  # CA runs only on controller nodes by default. Turn on `worker.nodePools[].clusterAutoscalerSupport.enabled` to
  # add nodes from a node pool to be where CA runs
  clusterAutoscaler:
    enabled: false

  # When enabled, Kubernetes rescheduler is deployed to the cluster controller(s)
  # This feature is experimental currently so may not be production ready
  rescheduler:
    enabled: false

# Experimental features will change in backward-incompatible ways
# experimental:
#   # Enable admission controllers
#   admission:
#     podSecurityPolicy:
#       enabled: true
#     denyEscalatingExec:
#       enabled: true
#   # Used to provide `/etc/environment` env vars with values from arbitrary CloudFormation refs
#   awsEnvironment:
#     enabled: true
#     environment:
#       CFNSTACK: '{ "Ref" : "AWS::StackId" }'
#   # Enable audit log for apiserver. Recommended when `rbac` is enabled.
#   auditLog:
#     enabled: true
#     maxage: 30
#     logpath: /dev/stdout
#   authentication:
#     # See https://kubernetes.io/docs/admin/authentication/#webhook-token-authentication
#     # for more information.
#     webhook:
#       enabled: true
#       cacheTTL: 1m0s
#       configBase64: base64-encoded-webhook-yaml
#   # Add predefined set of labels to the controller nodes
#   # The set includes names of launch configurations and autoscaling groups
#   awsNodeLabels:
#     enabled: true
#   # If enabled, instructs the controller manager to automatically issue TLS certificates to worker nodes via
#   # certificate signing requests (csr) made to the API server using the bootstrap token. It's recommended to
#   # also enable the rbac plugin in order to limit requests using the bootstrap token to only be able to make
#   # requests related to certificate provisioning.
#   # The bootstrap token is automatically generated in ./credentials/kubelet-tls-bootstrap-token.
#   tlsBootstrap:
#     enabled: true
#   # Enables the Node authorization + node restriction admission controller (requires Kubernetes 1.7.4+)
#   # Requires TLS bootstrapping to be enabled as well
#   nodeAuthorizer:
#     enabled: true
#   # This option has not yet been tested with rkt as container runtime
#   ephemeralImageStorage:
#     enabled: true
#   # When enabled it will grant sts:assumeRole permission to the IAM role for controller nodes.
#   # This is intended to be used in combination with .controller.managedIamRoleName. See #297 for more information.
#   kube2IamSupport:
#     enabled: true
#   # When enabled, `kubectl drain` is run when the instance is being replaced by the auto scaling group, or when
#   # the instance receives a termination notice (in case of spot instances)
#   nodeDrainer:
#     enabled: true
#     # Maximum time to wait, in minutes, for the node to be completely drained. Must be an integer between 1 and 60.
#     drainTimeout: 5
#   # Configure OpenID Connect token authenticator plugin in Kubernetes API server.
#   # For using Dex as a custom OIDC provider, please check "contrib/dex/README.md".
#   # WARNING: always use "https" for "issuerUrl", otherwise the Kubernetes API server will not start correctly.
#   oidc:
#     enabled: true
#     issuerUrl: "https://accounts.google.com"
#     clientId: "kubernetes"
#     usernameClaim: "email"
#     groupsClaim: "groups"
#
#   plugins:
#     rbac:
#       enabled: true
#
#   # When set to true this configures the k8s aws provider so that it doesn't modify every node's security group to include an additional ingress rule per an ELB created for a k8s service whose type is "LoadBalancer"
#   # It requires that the user has setup a rule that allows inbound traffic on kubelet ports
#   # from the local VPC subnet so that load balancers can access it.  Ref: https://github.com/kubernetes/kubernetes/issues/26670
#   disableSecurityGroupIngress: false
#
#   # Command line flag passed to the controller-manager. (default 40s)
#   # This is the amount of time which we allow running Node to be unresponsive before marking it unhealthy.
#   # Must be N times more than kubelet's nodeStatusUpdateFrequency (default 10s).
#   nodeMonitorGracePeriod: "40s"

# AWS Tags for cloudformation stack resources
#stackTags:
#  Name: "Kubernetes"
#  Environment: "Production"

# User-provided YAML map available in control-plane's stack-template.json
#customSettings:
#  key1: [ 1, 2, 3 ]
```
