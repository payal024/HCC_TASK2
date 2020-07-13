provider "aws" {
 region = "ap-south-1"
 profile = "payal24"
}
  resource "aws_key_pair" "payal286"{
  key_name = "payalkey"
  public_key = "ssh-rsa  AAAAB3NzaC1yc2EAAAADAQABAAABAQC7uFmedREmlz3/QHo0zFbMbjYeqKrykzRA1D6lT5UxbShwpYqYmvfl47zTPGVqirtHyGOVYcHlhtNEvia1qcNQqOHYB063FnlTLslwt1lET9k7SxWery1h7Xq79GUvg6uwPv6hEDbAoCvkwQ1Xml3PUS+HDQMru7FbUM1QQSDdctM+JL2iVYYUiK0A4y9G75ELnX/17wBOJjJt5HvoCFfAJbffpYMwmQcYFTdMshauJS/fmsMYN4kg7j6iHnYGPPEFIoMzMo+ujgFRBSraf9193UNU3u/zDK8+KoDm9GWJu3P9GRK778lgXSRuGyRSwMAU+y1HNr4UQ+vOzDMmy/l3"
}
 resource "aws_security_group" "tasksg2"{
 name = "tasksg2"
 description = "Allow TLS inbound traffic"
 vpc_id = "vpc-789f8210"
   
 
  ingress {
   description = "SSH"
   from_port = 22
   to_port = 22
   protocol = "tcp"
   cidr_blocks = [ "0.0.0.0/0" ]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "tasksg2"
  }
}
resource "aws_efs_file_system" "payalnfs" { creation_token = " payalnfs " 
tags = { 
Name = " payalnfs " 
} 
} 
resource "aws_efs_mount_target" "jajodia" { file_system_id="${aws_efs_file_system.payalnfs.id}” 
subnet_id = "${aws_subnet.jajodia.id}" 
security_groups = [ "${aws_security_group.nfs-sg.id}" 
] 
} 
resource "aws_subnet" "jajodia" { 
vpc_id = "${aws_security_group.nfssg.vpc_id}" 
availability_zone = "ap-south-1a" 
cidr_block = "172.31.48.0/20" } 
}

resource "aws_instance" "taskinst2" {
  ami           = "ami-0513891c972a0f002"
  instance_type = "t2.micro"
  availability_zone = "ap-south-1a"
  key_name      = "payal286"
  security_groups = [ "tasksg2" ]
  user_data = <<-EOF
  #! /bin/bash
sudo yum install httpd -y sudo systemctl start httpd sudo systemctl enable httpd
yum install -y amazon-efs-utils
apt-get -y install amazon-efs-utils
yum install -y nfs-utils
apt-get -y install nfs-common
file_system_id_1="${aws_efs_file_system.allownfs.id}"
efs_mount_point_1="/var/www/html"
mkdir -p "$efs_mount_point_1"
test -f "/sbin/mount.efs" && echo "$file_system_id_1:/
$efs_mount_point_1 efs tls,_netdev" >> /etc/fstab || echo
"$file_system_id_1.efs.ap-south-1.amazonaws.com:/
$efs_mount_point_1 nfs4
nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,
noresvport,_netdev 0 0" >> /etc/fstab
test -f "/sbin/mount.efs" && echo -e "\n[client-info]\nsource=liw" >>
/etc/amazon/efs/efs-utils.conf
mount -a -t efs,nfs4 defaults
sudo yum install git -y
cd /var/www/html
sudo yum install git -y
mkfs.ext4 /dev/xvdf1
mount /dev/xvdf1 /var/www/html
cd /var/www/html
git clone 
                
                
  EOF

  tags = {
    Name = "taskinst2"
  }
}

//Creating S3 bucket along with cloudfront
provider "aws" {
region = "ap-south-1"
profile = "payal24"
}
resource "aws_s3_bucket" "my-test-s3-terraform-bucket-payal24" {
bucket = "my-test-s3-terraform-bucket-payal24"
tags = {
Name = "my-test-s3-terraform-bucket-payal24"
}
}
resource "aws_cloudfront_distribution" "s3_distribution" {
origin {
domain_name ="${aws_s3_bucket.my-test-s3-terraform-bucket-payal24.bucket_regional_domain_name}"
origin_id ="${aws_s3_bucket.my-test-s3-terraform-bucket-payal24.id}"
}
enabled = true
is_ipv6_enabled = true
comment = "S3 bucket"
default_cache_behavior {
allowed_methods = ["DELETE", "GET", "HEAD", "OPTIONS",
"PATCH", "POST", "PUT"]
cached_methods = ["GET", "HEAD"]
target_origin_id ="${aws_s3_bucket.my-test-s3-terraform-bucket-payal24.id}"
forwarded_values {
query_string = false
cookies {
forward = "none"
}
}
viewer_protocol_policy = "allow-all"
min_ttl = 0
default_ttl = 3600
max_ttl = 86400
}
# Cache behavior with precedence 0
ordered_cache_behavior {
path_pattern = "/content/immutable/*"
allowed_methods = ["GET", "HEAD", "OPTIONS"]
cached_methods = ["GET", "HEAD", "OPTIONS"]
target_origin_id ="${aws_s3_bucket.my-test-s3-terraform-bucket-payal24.id}"
forwarded_values {
query_string = false
cookies {
forward = "none"
}
}
min_ttl = 0
default_ttl = 86400
max_ttl = 31536000
compress = true
viewer_protocol_policy = "redirect-to-https"
}
restrictions {
geo_restriction {
restriction_type = "whitelist"
locations = ["IN"]
}
}
tags = {
Environment = "production"
}
viewer_certificate {
cloudfront_default_certificate = true
}
depends_on = [
aws_s3_bucket.my-test-s3-terraform-bucket-payal24
]
