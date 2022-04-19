# Secrets

Secrets is an adapter to abstract the differences between different secrets backends so you can interact with them using a consistent interface.

## Interfaces
Interfaces exposed by Secrets are:

[SecretsStorage](https://github.com/infrahq/secrets/blob/main/secrets.go#L19) - Responsible for storing secrets, eg vault, AWS SSM, AWS Secrets Manager, Kubernetes Secrets, ENV variables, files, etc.

[SymmetricKeyProvider](https://github.com/infrahq/secrets/blob/main/secrets.go#L45) - an abstraction of a key service that provides key-encryption-as-a-service. Implementations will be able to generate keys for an app to encrypt and decrypt data, then be able to return the key again at a later time having only a key id. This is an abstraction of services like AWS KMS, with implementations for Vault, and a "Native" implementation in Go that uses SecretStorage as a storage backend. 

## Encryption

## Usage Example

using the vault secret provider

```go
  vault, err := secrets.NewVaultSecretProviderFromConfig(secrets.VaultConfig{
		TransitMount: "/transit",
		SecretMount:  "/secret",
    Token:        "xxx123",
    Namespace:    "namespace",
		Address:      "https://vault",
	})
	if err != nil {
    return err
  }

  err = vault.SetSecret("password", "fried chicken")
	if err != nil {
    return err
  }
  
  secret, err = vault.GetSecret("password")
 	if err != nil {
    return err
  }
  // secret == "fried chicken"
```

Using the native key provider with the file secret provider.

```go
  secretProvider := secrets.NewFileSecretProviderFromConfig(FileConfig{
		Path: os.TempDir(),
	})

	kp := secrets.NewNativeKeyProvider(secretProvider)

	key, err := kp.GenerateDataKey("rootKeyID")
	if err != nil {
    return err
  }

  // Save a copy of the encrypted key somewhere
  encryptedKey := key.Encrypted
  // We can later fetch it again from the service with:
  key, err = kp.DecryptDataKey("rootKeyID", encryptedKey)

	secretMessage := "toast"

	encrypted, err := secrets.Seal(key, []byte(secretMessage))
	if err != nil {
    return err
  }

	orig, err := secrets.Unseal(key, encrypted)
	if err != nil {
    return err
  }

  // orig == "toast"
```