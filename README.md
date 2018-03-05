# Node-config-cryptex

Use AWS KMS to securely encrypt your secrets and inject the decrypted values into your [node-config](https://github.com/lorenwest/node-config) configuration at run time.

Written as a plugin for [node-config](https://github.com/lorenwest/node-config)

## Purpose

Node-config is a great for configuration but doesn't provide any out of the box support for encrypting sensitive data (db passwords, api keys etc).  I also really like the approach taken by the [Cryptex library](https://github.com/TomFrost/Cryptex) when working with AWS KMS.

This library wraps the cryptex module for all management of KMS data keys as well as encryption/decryption of secrets.  It injects the resulting values into your config so you never have to store plaintext secrets into your repo

## Installation

```
npm install --save node-config-cryptex
npm install -g cryptex
```

## Configuration

1. In the AWS console create create a customer management key https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html.
2. Ensure that your application has permissions to encrypt/decrypt using this key.  You may also need to giver permission to your developer's IAM profiles if they need to run your application locally.
3. Create a data key:
    ```
    aws kms generate-data-key-without-plaintext \
      --key-id alias/YOUR_KEY_ALIAS \
      --key-spec AES_256 \
      --output text \
      --query CiphertextBlob
    ```
4. Create a cryptex config as documented [here](https://github.com/TomFrost/Cryptex/blob/master/README.md#5-save-your-secrets). *NOTE: don't put your secrets in the secrets section of the cryptext config. We'll put these inline with the rest of your config*
5. Encrypt a secret -
    If you installed Cryptex globally, you'll have a CLI tool called `cryptex` that can encrypt and decrypt your keys according to your `cryptex.json`.

    ```
    $ cryptex encrypt mypassword
    Q+JfrQS5DtSjqWHu1oO4HqctA2hVw4VhaDQfBCuvO8U=
    ```
6. Add the encrypted value into your config file with the prefix `CRYPT:` ex:
    ```yml
    db:
        user: db_user
        password: CRYPT:Q+JfrQS5DtSjqWHu1oO4HqctA2hVw4VhaDQfBCuvO8U=
    ```

## Usage
All secrets **MUST be loaded before config.get is called**. Once `config.get` is called the value becomes immutable so if we call `config.get` we can't update the encrypted value with the decrypted one.

The easiest way is calling config.loadSecrets before your app entry point is required.

```js
const config = require("node-config-cryptex");

config.loadSecrets().then(() => {
    require("./app.js");
});
```

Or if you're into async

```js
const config = require("node-config-cryptex");

await config.loadSecrets();
require("./app.js");
```

## How it works

