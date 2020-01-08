# Installing GitHub Enterprise Server on AWS

<div class="3D&quot;lead-mktg&quot;">

To install GitHub Enterprise Server on Amazon Web Services (AWS), you must launch an Amazon Elastic Compute Cloud (EC2) instance and create and attach a separate Amazon Elastic Block Store (EBS) data volume.

</div>

*   [Prerequisites]

*   [Hardware considerations]

*   [Determining the instance type]

*   [Selecting the GitHub Enterprise Server AMI]

*   [Creating a security group]

*   [Creating the GitHub Enterprise Server instance]

*   [Configuring the GitHub Enterprise Server instance]

### Prerequisites

*   You must have a GitHub Enterprise license file. To download an existing = license file or request a trial license, visit [enterprise.github.com]. For more information, see= "[Managing your GitHub Enterprise Server license]."

*   You must have an AWS account capable of launching EC2 instances and crea= ting EBS volumes. For more information, see the [Amazon Web Services website].

*   Most actions needed to launch your GitHub Enterprise Server instance may= also be performed using the AWS management console. However, we recommend = installing the AWS command line interface (CLI) for initial setup. Examples= using the AWS CLI are included below. For more information, see Amazon's g= uides "[Working with the AWS Management Console]" and "[What is the AWS Command Line Interface]."

This guide assumes you are familiar with the following AWS concepts:

*   [Launching EC2 Instances]
*   [Managing EBS Volumes]
*   [Using Security Groups] (For managing network access = to your instance)
*   [Elastic IP Addresses (EIP)] (Strongly recommended = for production environments)
*   [EC2 and Virtual Private Cloud] (If you plan to launch into a Virt= ual Private Cloud)

### [Hardware considerations]

GitHub Enterprise Server requires a persistent data disk separate from t= he root disk. For more information, see "[System overview]."

We recommend different hardware configurations based on the number of us= er licenses used in your GitHub Enterprise Server instance.

<table>

<thead>

<tr>

<th align="3D&quot;center&quot;">User licenses</th>

<th align="3D&quot;center&quot;">vCPUs</th>

<th align="3D&quot;center&quot;">Memory</th>

<th align="3D&quot;center&quot;">Attached storage</th>

<th align="3D&quot;center&quot;">Root storage</th>

</tr>

</thead>

<tbody>

<tr>

<td align="3D&quot;center&quot;">Trial, demo, or 10 light users</td>

<td align="3D&quot;center&quot;">2</td>

<td align="3D&quot;center&quot;">16 GB</td>

<td align="3D&quot;center&quot;">100 GB</td>

<td align="3D&quot;center&quot;">200 GB</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">10-3000</td>

<td align="3D&quot;center&quot;">4</td>

<td align="3D&quot;center&quot;">32 GB</td>

<td align="3D&quot;center&quot;">250 GB</td>

<td align="3D&quot;center&quot;">200 GB</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">3000-5000</td>

<td align="3D&quot;center&quot;">8</td>

<td align="3D&quot;center&quot;">64 GB</td>

<td align="3D&quot;center&quot;">500 GB</td>

<td align="3D&quot;center&quot;">200 GB</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">5000-8000</td>

<td align="3D&quot;center&quot;">12</td>

<td align="3D&quot;center&quot;">96 GB</td>

<td align="3D&quot;center&quot;">750 GB</td>

<td align="3D&quot;center&quot;">200 GB</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">8000-10000+</td>

<td align="3D&quot;center&quot;">16</td>

<td align="3D&quot;center&quot;">128 GB</td>

<td align="3D&quot;center&quot;">1000 GB</td>

<td align="3D&quot;center&quot;">200 GB</td>

</tr>

</tbody>

</table>

These are minimum recommendations. More resources may be required depend= ing on your usage, such as user activity and selected integrations. When in= creasing CPU resources, it's recommended to add at least 6.5 GB of memory f= or each CPU (up to 16 CPUs) added to your GitHub Enterprise Server instance= . For more information, see "[Increasing CPU or memory resources]."

<div class="3D&quot;extended-markdown" note="" border="" rounded-1="" mb-4="" p-3="" border-blue="bg-blue-light" f5"="">

**Note:** The root disk can be resized by building a new ap= pliance or using an existing appliance. For more information, see "[Increasing storage capacity](=3D"https://help.github.com/enterprise/2.19/admin/guides/installation/incre=)."

</div>

### [Determining the instance type]

Before launching your GitHub Enterprise Server instance on AWS, you'll n= eed to determine the type of virtual machine that best fits the needs of yo= ur organization.

#### [Supported instance types]

GitHub Enterprise Server is supported on the following EC2 instance type= s. For more information, see [the AWS EC2 instance type overview page].

<table>

<thead>

<tr>

<th>EC2 instance type</th>

<th>Model</th>

</tr>

</thead>

<tbody>

<tr>

<td>C3</td>

<td>c3.2xlarge, c3.4xlarge, c3.8xlarge</td>

</tr>

<tr>

<td>C4</td>

<td>c4.2xlarge, c4.4xlarge, c4.8xlarge</td>

</tr>

<tr>

<td>C5</td>

<td>c5.large, c5.xlarge, c5.2xlarge, c5.4xlarge, c5.9xlarge, c5.18xlarge</td>

</tr>

<tr>

<td>M3</td>

<td>m3.xlarge, m3.2xlarge</td>

</tr>

<tr>

<td>M4</td>

<td>m4.xlarge, m4.2xlarge, m4.4xlarge, m4.10xlarge, m4.16xlarge</td>

</tr>

<tr>

<td>M5</td>

<td>m5.large, m5.xlarge, m5.2xlarge, m5.4xlarge, m5.12xlarge, m5.24xlarge</td>

</tr>

<tr>

<td>R4</td>

<td>r4.large, r4.xlarge, r4.2xlarge, r4.4xlarge, r4.8xlarge, r4.16xlarge</td>

</tr>

<tr>

<td>R5</td>

<td>r5.large, r5.xlarge, r5.2xlarge, r5.4xlarge, r5.12xlarge, r5.24xlarge</td>

</tr>

<tr>

<td>X1</td>

<td>x1.16xlarge, x1.32xlarge</td>

</tr>

</tbody>

</table>

#### [Recommended instance types]

Based on your user license count, we recommend the following instance types.

<table>

<thead>

<tr>

<th align="3D&quot;center&quot;">User license</th>

<th align="3D&quot;center&quot;">Recommended type</th>

</tr>

</thead>

<tbody>

<tr>

<td align="3D&quot;center&quot;">Trial, demo, or 10 light users</td>

<td align="3D&quot;center&quot;">r4.large</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">10 - 3000</td>

<td align="3D&quot;center&quot;">r4.xlarge</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">3000 - 5000</td>

<td align="3D&quot;center&quot;">r4.2xlarge</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">5000 - 8000</td>

<td align="3D&quot;center&quot;">r4.4xlarge</td>

</tr>

<tr>

<td align="3D&quot;center&quot;">8000 - 10000+</td>

<td align="3D&quot;center&quot;">r4.8xlarge</td>

</tr>

</tbody>

</table>

<div class="3D&quot;extended-markdown" warning="" border="" rounded-1="" mb-4="" p-3="" border-re="d" bg-red-light="" f5"="">

**Note:** You can always scale up your CPU or memory by res= izing your instance. However, because resizing your CPU or memory requires = downtime for your users, we recommend over-provisioning resources to account for scale.

</div>

### [Selecting t= he GitHub Enterprise Server AMI]

You can select an Amazon Machine Image (AMI) for GitHub Enterprise Serve= r using the GitHub Enterprise Server portal or the AWS CLI.

AMIs for GitHub Enterprise Server are available in the AWS GovCloud (US-= East and US-West) region. This allows US customers with specific regulatory= requirements to run GitHub Enterprise Server in a federally compliant clou= d environment. For more information on AWS's compliance with federal and ot= her standards, see [AWS's Gov= Cloud (US) page] and [AWS' compliance page]("https://aws.amazon.com/compliance/").

#### Using the GitHub Enterprise Server portal to select an AMI<

1.  Navigate to [the GitHu= b Enterprise Server download page].

2.  Click **Get the latest release of GitHub Enterprise Server.**

***   In the Select your platform drop-down menu, click **Amazon Web Services**.

    *   In the Select your AWS region drop-down menu, choose your desired region= .

    *   Take note of the AMI ID that is displayed.

    **

**

#### [Using the AWS CLI to select= an AMI]

1.  Using the AWS CLI, get a list of GitHub Enterprise Server images publish= ed by GitHub's AWS owner IDs (`025577942450` for GovCloud, and <= code>895557238572 for other regions). For more information, see "[describe-images](3D"http://docs.aws.amazon.com/cli/latest/reference/ec2/describe-images=)" in the AWS documentation.

        aws ec2 describe-images \
        --owners OWNER ID \
        --query 'sort_by(Images,&Name)[*].{Name:Name,ImageID:ImageId}' \
        --output=3Dtext

2.  Take note of the AMI ID for the latest GitHub Enterprise Server image.<= /li>

### [Creating a security group]

If you're setting up your AMI for the first time, you will need to creat= e a security group and add a new security group rule for each port in the t= able below. For more information, see the AWS guide "[Using Security Groups=]."

1.  Using the AWS CLI, create a new security group. For more information, see "[create-security-group](3D"http://docs.aws.amazon.com/cli/latest/reference/ec2/create-se=)" in the AWS documentation.

        $ aws ec2 create-security-group --=
        group-name SECURITY_GROUP_NAME --description "SECURITY GROUP D=
        ESCRIPTION"

2.  Take note of the security group ID (`sg-xxxxxxxx`) of your newly created security group.

3.  Create a security group rule for each of the ports in the table below. F= or more information, see "[authorize-security-group-ingress](3D"http://docs.aws.amazon.com/cli/latest/=)" in the AWS documentation.

        $ aws ec2 authorize-security-gro=
        up-ingress --group-id SECURITY_GROUP_ID --protocol PROTOCOL --port PORT_NUMBER --cidr SOURCE IP RANGE

    _

    This table identifies what each port is used for.

    <table>

    <thead>

    <tr>

    <th>Port</th>

    <th>Service</th>

    <th>Description</th>

    </tr>

    </thead>

    <tbody>

    <tr>

    <td>22</td>

    <td>SSH</td>

    <td>Git over SSH access. Clone, fetch, and push operations to public/privat= e repositories supported.</td>

    </tr>

    <tr>

    <td>25</td>

    <td>SMTP</td>

    <td>SMTP with encryption (STARTTLS) support.</td>

    </tr>

    <tr>

    <td>80</td>

    <td>HTTP</td>

    <td>Web application access. _All requests are redirected to the HTTPS po= rt when SSL is enabled.</td>

    </tr>

    <tr>

    <td>122</td>

    <td>SSH</td>

    <td>Instance shell access. _The default SSH port (22) is dedicated to ap= plication git+ssh network traffic.</td>

    </tr>

    <tr>

    <td>161/UDP</td>

    <td>SNMP</td>

    <td>Required for network monitoring protocol operation.</td>

    </tr>

    <tr>

    <td>443</td>

    <td>HTTPS</td>

    <td>Web application and Git over HTTPS access.</td>

    </tr>

    <tr>

    <td>1194/UDP</td>

    <td>VPN</td>

    <td>Secure replication network tunnel in high availability configuration.</td>

    </tr>

    <tr>

    <td>8080</td>

    <td>HTTP</td>

    <td>Plain-text web based Management Console. _Not required unless SSL is= disabled manually._</td>

    </tr>

    <tr>

    <td>8443</td>

    <td>HTTPS</td>

    <td>Secure web based Management Console. _Required for basic installatio= n and configuration.</td>

    </tr>

    <tr>

    <td>9418</td>

    <td>Git</td>

    <td>Simple Git protocol port. Clone and fetch operations to public reposito= ries only. Unencrypted network communication.</td>

    </tr>

    </tbody>

    </table>

    _

_

### [Creating the GitHub Enterprise Server instance]

To create the instance, you'll need to launch an EC2 instance with your = GitHub Enterprise Server AMI and attach an additional storage volume for yo= ur instance data. For more information, see "[Hardware considerations](3D"https://help.github=)."

<div class="3D&quot;extended-markdown" note="" border="" rounded-1="" mb-4="" p-3="" border-blue="bg-blue-light" f5"="">

**Note:** You can encrypt the data disk to gain an extra level of security and ensure that any data you write to your instance is protected. There is a slight performance impact when using encrypted disks. If = you decide to encrypt your volume, we strongly recommend doing so **before** starting your instance for the first time. For more information, see the [Amazon guide on EBS encryption](3D"http://docs.aws.amazon.com/AWSEC2/=).

</div>

<div class="3D&quot;extended-markdown" warning="" border="" rounded-1="" mb-4="" p-3="" border-re="d" bg-red-light="" f5"="">

**Warning:** If you decide to enable encryption after you'v= e configured your instance, you will need to migrate your data to the encrypted volume, which will incur some downtime for your users.

</div>

#### [Launching an EC2 instance]

In the AWS CLI, launch an EC2 instance using your AMI and the security g= roup you created. Attach a new block device to use as a storage volume for = your instance data, and configure the size based on your user license count= . For more information, see "[run-instances](3D"http://docs.aws.amazon.com/cli/late=)" in the AWS document= ation.

_

<pre>_`aws ec2 run-instances \
  --security-group-ids _SECURITY_GROUP_ID_ \
  --instance-type _INSTANCE_TYPE_ \
  --image-id _AMI_ID_ \
  --block-device-mappings '[{"DeviceName":"/dev/xvdf","Ebs":{"VolumeSize":<=
em>SIZE`_`,"VolumeType":"_TYPE_"}}]' \
  --region _REGION_ \
  --ebs-optimized`</pre>

#### Allocating an Elastic IP and associating it with= the instance

If this is a production instance, we strongly recommend allocating an Elastic IP (EIP) and associating it with the instance before proceeding to GitHub Enterprise Server configuration. Otherwise, the public IP address of the instance will not be retained after instance restarts. For more information, see "[Allocating an Elastic IP Address](3D"http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ela=)" and "[Associating an Elastic IP Address with a Running Instance](3D"http://docs.aws.amazon.com=)" in the Amazon documentation.

Both primary and replica instances should be assigned separate EIPs in p= roduction High Availability configurations. For more information, see "Configuring GitHub Enterprise Server for High Availability."

### [Configuring the GitHub Enterprise Server instance]

1.  Copy the virtual machine's public DNS name, and paste it into a web browser.

2.  At the prompt, upload your license file and set a management console password. For more information, see "[Managing your GitHub Enterprise Server license]."

3.  In the [Management Console], configure and save your desired settings. For more information, see "[Configuring the GitHub Enterprise Server appliance]."

4.  The instance will restart automatically.

5.  Click **Visit your instance**.

**</article>

</main>
