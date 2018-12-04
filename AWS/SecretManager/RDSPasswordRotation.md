# RDS Password Rotation

## Network Requirements

For RDS created inside a VPC the Lambda script is created with no Security
Group. A group needs to be created and applied with the following access:

  1) Outbound `TCP 443 HTTPS` to `0.0.0.0/0` for access to secret manager endpoint.
  2) Outbound `TCP 3306 MySQL` (or relevant database port) to access database.

The security group on the Database must equally have a corresponding ingress rule
to allow the Lambda endpoint traffic to be received.

As this security group must be added after the initial attempt of rotation will fail.

## Clearing stalled rotation

1) Remove the AWSPENDING label from the secret version, you can do so with a command like the following:

    ```bash
    aws secretsmanager update-secret-version-stage \
    --secret-id ARN-OF-YOUR-SECRET \
    --version-stage AWSPENDING \
    --remove-from-version-id 62e34cc2-0620-4885-ba4b-290c27053c53
    ```

    (In the command above, substitute the version ID with that of the version that currently has the AWSPENDING label).

2) Call the rotate-secret API again specifying the same version id of the secret that has the AWSPENDING label.

    ```bash
    aws secretsmanager rotate-secret
    --secret-id ARN-OF-YOUR-SECRET 
    --client-request-token 62e34cc2-0620-4885-ba4b-290c27053c53
    ```

    (In the command above, substitute the client request token value with the ID of the version that currently has the AWSPENDING label).

    Once you do this, try to rotate the secret again and check for the results with describe-secret. You can also check the results of the Lambda execution in CloudWatch logs. [1].
