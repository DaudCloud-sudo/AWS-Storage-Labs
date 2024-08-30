# AWS-Storage-Labs

# Creating an S3 Bucket and Modifying the EC2 Instance

In this lab, we will create an S3 bucket to store employee photos, upload an object to it, modify the bucket policy, and launch an EC2 instance with updated user data to use the S3 bucket. Finally, we will stop the EC2 instance to prevent future costs.

## Table of Contents
- [Task 1: Creating an S3 Bucket](#task-1-creating-an-s3-bucket)
- [Task 2: Uploading a Photo](#task-2-uploading-a-photo)
- [Task 3: Modifying the S3 Bucket Policy](#task-3-modifying-the-s3-bucket-policy)
- [Task 4: Modifying the Application to Use the S3 Bucket](#task-4-modifying-the-application-to-use-the-s3-bucket)
- [Task 5: Deleting the Object from the S3 Bucket](#task-5-deleting-the-object-from-the-s3-bucket)
- [Task 6: Stopping Your EC2 Instance](#task-6-stopping-your-ec2-instance)

---

## Task 1: Creating an S3 Bucket

1. Log in to the AWS Management Console with your Admin user.
2. In the search box, enter **S3** and open the Amazon S3 console.
3. Choose **Create bucket**.
4. For **Bucket name**, enter `employee-photo-bucket-<your initials>-<unique number>`.

    **Example:**
    ```
    employee-photo-bucket-daudcloud-42
    ```
5. Choose **Create bucket**.

![image](https://github.com/user-attachments/assets/8f631b8a-9cf0-4a99-a044-268f9b3e5a82)

## Task 2: Uploading a Photo

1. Open the details of your newly created bucket by choosing the bucket name.
2. Choose **Upload**.
3. Choose **Add files**.
4. Select a photo of your choice from your computer and choose **Open**.
5. Choose **Upload**.
6. You should see **Upload succeeded** in green at the top.
7. Choose **Close**.

![image](https://github.com/user-attachments/assets/f465032c-a18c-40f4-b226-58dc7af0f5ed)

## Task 3: Modifying the S3 Bucket Policy

1. Go to the **Permissions** tab of your S3 bucket.
2. Scroll down to **Bucket policy** and choose **Edit**.
3. In the box, paste the following policy:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "AllowS3ReadAccess",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::<INSERT-ACCOUNT-NUMBER>:role/S3DynamoDBFullAccessRole"
                },
                "Action": "s3:*",
                "Resource": [
                    "arn:aws:s3:::<INSERT-BUCKET-NAME>",
                    "arn:aws:s3:::<INSERT-BUCKET-NAME>/*"
                ]
            }
        ]
    }
    ```

    Replace `<INSERT-ACCOUNT-NUMBER>` with your account number and `<INSERT-BUCKET-NAME>` with your bucket name.
   
Like this:
![image](https://github.com/user-attachments/assets/06f035b4-98bf-48f3-b6fc-790eb6fb58b7)

4. Choose **Save changes**.

## Task 4: Modifying the Application to Use the S3 Bucket

1. In the AWS Management Console, search for **EC2** and open the service.
2. In the navigation pane, under **Instances**, choose **Instances**.
3. Select the **employee-directory-app** instance (should be in the **Stopped** state).
4. Choose **Actions** > **Image and templates** > **Launch more like this**.
5. For **Name**, append `-s3` to the value.

    **Example:**
    ```
    employee-directory-app-s3
    ```

6. For **Key pair name**, select **app-key-pair**.
7. Under **Network settings** and **Auto-assign Public IP**, choose **Enable**.
8. Scroll down and expand **Advanced Details**.
9. In the **User data** box, update the values for the `PHOTOS_BUCKET` and (if needed) the `AWS_DEFAULT_REGION` variable.

    ```bash
    #!/bin/bash -ex
    wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
    unzip FlaskApp.zip
    cd FlaskApp/
    yum -y install python3-pip
    pip install -r requirements.txt
    yum -y install stress
    export PHOTOS_BUCKET=employee-photo-bucket-daudcloud-42
    export AWS_DEFAULT_REGION=<INSERT REGION HERE>
    export DYNAMO_MODE=on
    FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
    ```
![image](https://github.com/user-attachments/assets/0131c704-8634-406b-a978-32da64ec4541)

10. Choose **Launch instance**.
11. Choose **View all instances**. Wait for the instance state to change to **Running** and the status check to change to **2/2 checks passed**.

![image](https://github.com/user-attachments/assets/8ecd4e96-9d86-4758-9a2c-983ea4354d2a)

13. Copy the **Public IPv4 address** and paste it into a new browser window, making sure to use HTTP instead of HTTPS.
14. You should see an **Employee Directory** placeholder.

For detailed EC2 setup instructions, please refer to [AWS Compute Labs](https://github.com/DaudCloud-sudo/AWS-Compute-Labs).

15. Once you upload a photo from the app, you should be able to view it in the S3 bucket.

![image](https://github.com/user-attachments/assets/0c54fbcd-1d09-4430-9745-b8302bb76ff7)

## Task 5: Deleting the Object from the S3 Bucket

1. Open the Amazon S3 console.
2. Open the bucket details by choosing the **employee-photo-bucket-** link.
3. Select the check box for your object.
4. Choose **Delete** and confirm the deletion by entering **permanently delete**.
5. Choose **Delete objects** and then choose **Close**.

## Task 6: Stopping Your EC2 Instance

1. Return to the Amazon EC2 console.
2. If needed, in the navigation pane, choose **Instances**.
3. Select the check box for the **employee-directory-app-s3** instance.
4. Choose **Instance state** > **Stop instance**.
5. In the dialog box, choose **Stop**.


**Note:** This guide is based on the AWS SAA preparation course lab. Ensure you terminate all resources to avoid incurring costs.
