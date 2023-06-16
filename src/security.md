# Security

## ENV File

To manage configuration settings for our apps, especially when deploying them to different environments, we take advantage of using env files. This also helps us avoid hardcoding values in our code, particularly in the frontend, where the code runs on the client's browser.

We put our variable names in an `.env.example` file. This serves as a guide for developers, letting them know what are the variables used in our configuration.

`.env.example`

```js
APP_ENV=local
APP_NAME=
PORT=4000
JWT_SECRET=
DB_HOST=
DB_USERNAME=
DB_PASSWORD=
DB_NAME=
```

When it comes to security, we should include the env file in the `.gitignore` file to avoid it being included in the codebase.

We also use `AWS Systems Manager Parameter Store` to store environment data for our staging and production environments. As part of our deployment pipeline, we have included a script that retrieves these values using the AWS CLI and generates a new .env file for each deployment. This approach helps ensure that our deployment process is reproducible and that sensitive information is kept secure.

```yml
# Get parameters from AWS and put them into the .env file
aws ssm get-parameter --with-decryption --name $PARAMATER --region $REGION | jq -r '.Parameter.Value' > $WEB_DIR/.env
```

<br />

## JSON Web Tokens (JWT)

We chose to use JWT for our authentication and authorization.

When a user logs in, the server generates a JWT and sends it to the client. The client then includes the JWT in subsequent requests to the server as an authorization header. The server verifies the signature and decodes the payload to determine if the user is authorized to perform the requested action.

JWTs are considered safe because they are digitally signed using a secret key. This means that the server can verify that the token has not been tampered with and that it was created by a trusted party. Additionally, since the token contains all the necessary information, the server does not need to store session data, making it less vulnerable to attacks that target session data.

Here's a sample implementation:

```ts
export function generateAccessToken(user: UserAuth): string {
  return jwt.sign(user, process.env.JWT_SECRET as string, { expiresIn: "1d" });
}
```

This function can be used in a login process. This generates a JWT and sends it to the frontend upon successful authentication. The JWT contains information about the user, such as user ID or role, and is digitally signed using a secret key. We also specify an expiration time in the _options_ parameter.

We will then get the token from the cookies in the request and use our secret key again to verify if the user is authorized to make their request. The _verify_ function returns the signed data (_user_) as an output.

```ts
const token = req.cookies.token || '';
jwt.verify(token, process.env.JWT_SECRET as string, (err: any, user: any) => {
// proceed
}
```

To authenticate the requests sent from the frontend app, we include the cookies in the request headers by passing the `withCredentials` option.

```js
const response = await API.post('/users/check', {}, { withCredentials: true });
```
<br />

### Local Storage

We make sure **not** to store the tokens in the Local Storage. This is a common mistake. An attacker can get a copy of JWT to make requests on behalf of the user. Instead, we should store them in server-side cookies. This [article](https://www.rdegges.com/2018/please-stop-using-local-storage/) explains it better.

<br />

## Other security policies

There are some other policies we enforce, including:

1. Code artifacts in S3:

- Encryption of data at rest: Ensure that all data stored in S3 is encrypted using AWS S3 Server-Side Encryption (SSE) or client-side encryption.
- Access control: Configure appropriate S3 bucket policies and IAM roles to restrict access to the data stored in S3, and limit access to only authorized users and applications.

2. User authentication:

- Secure login form: Implement a secure login form using HTTPS to ensure that user credentials are transmitted securely.
- Password policies: Enforce strong password policies, such as minimum length, complexity, and expiration.

3. API and Database communication:

- Encryption of data in transit: Use SSL/TLS encryption to secure communication between the API and Database, and to protect against eavesdropping and man-in-the-middle (MITM) attacks.
- Secure connection: Ensure that the API and Database are configured to use secure connections, and that only authorized users and applications have access.

4. Data in RDS (PlanetScale):

- Encryption of data at rest: Ensure that all data stored in RDS is encrypted using AWS RDS encryption or third-party encryption tools.
- Access control: Use RDS security groups and IAM roles to restrict access to the data stored in RDS, and limit access to only authorized users and applications.
- Data backup and recovery: Implement regular data backups and a disaster recovery plan to ensure that data can be recovered in case of a security breach or data loss.

<br />

---