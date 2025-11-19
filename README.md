## PHP Image Upload to S3 with IAM Role and EC2 DB Server

## Overview
This project demonstrates uploading images from a PHP + Nginx application hosted on an EC2 instance to Amazon S3 using an IAM Role for secure access. The metadata is stored in a MySQL database hosted on a separate EC2 DB Server.

## Architecture
```
User → EC2 App Server (PHP + Nginx) → S3 Bucket
→ EC2 DB Server (MySQL) + Database 
IAM Role → Grants S3 access
```

## Features
- PHP 8.3 + Nginx
- Image upload to S3
- MySQL on EC2 DB Server
- IAM Role-based access (no access keys)
- GitHub-ready documentation

## IAM Role Setup
1. Go to IAM → Roles → Create Role
2. Select **EC2** as trusted service
3. Attach **AmazonS3FullAccess**
4. Name the role (example: `S3UploadRole`)
5. Attach the role to EC2 App Server

## S3 Setup
Create an S3 bucket with ACL ENABLED and give public access to objects .

## EC2 App Server Setup
install required services in App server
```
sudo apt update
sudo apt install nginx mariadb-server php php8.3-fpm php8.3-mysql -y
sudo service nginx restart
sudo service php8.3-fpm restart
cd /var/www/html/

```
Allow http send request inside SG of App server

```
sudo nano test.php
sudo service nginx restart
sudo nano form.html
sudo nano upload.php

```
## Create Uploads Directory
```
sudo mkdir uploads
sudo chmod -R 777 uploads/
```
## Install AWS-SDK for php 
```
sudo curl -sS https://getcomposer.org/installer | sudo php
sudo mv composer.phar /usr/local/bin/composer
sudo ln -s /usr/local/bin/composer /usr/bin/composer
sudo composer require aws/aws-sdk-php
sudo composer require aws/aws-sdk-php --ignore-platform-req=ext-curl --ignore-platform-req=ext-simplexml

```

## EC2 DB Server Setup
Install MySQL and create database + table.
```
sudo apt update
sudo apt install mariadb-server -y
sudo mysql
alter user 'root'@'localhost' identified by 'Pass@123';
create database facebook;
create user 'sam'@'172.31.19.91' identified by 'sam$123';
Grant all privileges on facebook.* to 'sam'@'172.31.19.91';
flush privileges;
use facebook;
create table posts(id int primary key auto_increment, name varchar(50), url varchar(100));
```

## Frontend (form.html)

```
<!DOCTYPE html>
<html>
<body>
  <form action="upload.php" method="post" enctype="multipart/form-data">
Name:<input type="text" id="name" name="name">
            Select image to upload:
<input type="file" name="anyfile" id="anyfile">
<input type="submit" value="Upload Image" name="submit">
</form>
```

## Backend (upload.php)
Handles uploading file to S3 and storing metadata in DB.
```
<?php
require 'vendor/autoload.php';
use Aws\S3\S3Client;
// Instantiate an Amazon S3 client.
$s3Client = new S3Client([
'version' => 'latest',
'region'  => 'us-east-1'
//'credentials' => [
//'key'    => ' AKIAR3KPMGMSXLXA5HXI ',     //Add your access key here
//'secret' => ' yH0f0bFXWZYubCWGI8uy+BObnbqZuam1h2H2bXov'  //Add your secret key here
//]
]);
// Check if the form was submitted
if($_SERVER["REQUEST_METHOD"] == "POST"){
// Check if file was uploaded without errors
if(isset($_FILES["anyfile"]) && $_FILES["anyfile"]["error"] == 0){
$allowed = array("jpg" => "image/jpg", "jpeg" => "image/jpeg", "gif" => "image/gif", "png" => "i>$filename = $_FILES["anyfile"]["name"];
$filetype = $_FILES["anyfile"]["type"];
$filesize = $_FILES["anyfile"]["size"];
// Validate file extension
$ext = pathinfo($filename, PATHINFO_EXTENSION);
 
if(!array_key_exists($ext, $allowed)) die("Error: Please select a valid file format.");
// Validate file size - 10MB maximum
$maxsize = 10 * 1024 * 1024;
if($filesize > $maxsize) die("Error: File size is larger than the allowed limit.");
// Validate type of the file
if(in_array($filetype, $allowed)){
// Check whether file exists before uploading it
if(file_exists("uploads/" . $filename)){
echo $filename . " is already exists.";
} else{
if(move_uploaded_file($_FILES["anyfile"]["tmp_name"], "uploads/" . $filename)){
$bucket = 'mybucket-19-11-25';              //Add your bucket name here
$file_Path = __DIR__ . '/uploads/'. $filename;
$key = basename($file_Path);
try {
$result = $s3Client->putObject([
'Bucket' => $bucket,
'Key'    => $key,
'Body'   => fopen($file_Path, 'r'),
'ACL'    => 'public-read', // make file 'public'
]);
echo "Image uploaded successfully. Image path is: ". $result->get('ObjectURL');
 
$urls3= $result->get('ObjectURL') ;
//$cfurl= str_replace("https://vsb17.s3.amazonaws.com","https://d1lspta3i8hms0.cloudfront.net", >//echo $cfurl;
$name=$_POST["name"];
$servername = "172.31.69.87";
$username = "swati";
$password = "Swati@123";
$dbname = "facebook";
// Create connection
$conn = mysqli_connect($servername, $username, $password, $dbname);
// Check connection
if (!$conn) {
  die("Connection failed: " . mysqli_connect_error());
}
$sql = "INSERT INTO posts(name,url) VALUES('$name','$urls3')";
if (mysqli_query($conn, $sql)) {
  echo "New record created successfully";
} else {
  echo "Error: " . $sql . "<br>" . mysqli_error($conn);
}
 
mysqli_close($conn);
 
 
} catch (Aws\S3\Exception\S3Exception $e) {
echo "There was an error uploading the file.\n";
echo $e->getMessage();
}
echo "Your file was uploaded successfully.";
}else{
echo "File is not uploaded";
}
}
} else{
echo "Error: There was a problem uploading your file. Please try again.";
}
} else{
echo "Error: " . $_FILES["anyfile"]["error"];
}
}
?>
```
## Nginx Configuration
Configure PHP processing and document root.

---
**Author**: Sudarshan Mane
---
**LinkedIn**: https://www.linkedin.com/in/sudarshan-mane-504a59294
  
