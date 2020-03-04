# Kubernetes Secret Encryption.

Kubernetes supports the ability to encrypt secret data at rest.
This means that secrets are encrypted so that are never stored on disc in plain text.
This feature is important for security, but in order to use it we need to provide Kubernetes with an encryption key. 
We will generate an encryption key and put it into a configuration file. We will then copy that file to our kubernetes controller servers.

Generate the Kubernetes Data encrpytion config file containing the encrpytion key:

````
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

cat > encryption-config.yaml << EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
````

Copy the file to both controller servers:

````
scp encryption-config.yaml user@<controller 1 public ip>:~/
scp encryption-config.yaml user@<controller 2 public ip>:~/
````
