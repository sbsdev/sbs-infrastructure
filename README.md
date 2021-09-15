# SBS infrastructure

[Ansible playbooks][] to install a [Daisyproducer][], a [Daisyproducer2][], a [Kati][] or a [Madras2][] server.

## Deployment

### Daisyproducer

Put the needed debs into `files`. Adapt the version numbers in
`host_vars/xmlp-test` and `host_vars/xmlp`. To deploy to the test
server run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt daisyproducer.yml
```

To deploy to the production server run

```
ansible-playbook -i production -K --vault-password-file .vault_pass.txt daisyproducer.yml
```

### Daisyproducer2

Create a release in [Daisyproducer2][] with `lein release`. This
triggers the [release github
action](https://github.com/sbsdev/daisyproducer2/blob/main/.github/workflows/upload-release-asset.yml)
which builds the uberjar and uploads it to the [github release
page](https://github.com/sbsdev/daisyproducer2/releases). Then adapt
the version number in `roles/daisyproducer2/defaults/main.yml`. To
deploy to the test server run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt daisyproducer2.yml
```

To deploy to the production server run

```
ansible-playbook -i production -K --vault-password-file .vault_pass.txt daisyproducer2.yml
```

### Kati

Create a release in [Kati][] with `lein release`. This triggers the
[release github
action](https://github.com/sbsdev/catalog/blob/master/.github/workflows/upload-release-asset.yml)
which builds the uberjar and uploads it to the [github release
page](https://github.com/sbsdev/catalog/releases). Then adapt the
version number in `roles/kati/defaults/main.yml`. To deploy to the
test server run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt kati.yml
```

To deploy to the production server run

```
ansible-playbook -i production -K --vault-password-file .vault_pass.txt kati.yml
```

### Madras2

To deploy to the madras test server run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt madras2.yml
```

### Pipeline

To deploy to the pipeline test server run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt pipeline.yml
```

## Edit Passwords

```
ansible-vault edit --vault-password-file .vault_pass.txt vars/passwords.yml
```

## License

Copyright (c) 2014 [Swiss Library for the Blind, Visually Impaired and Print Disabled](http://www.sbs.ch/).

Licensed under the [MIT License](./LICENSE).

[Ansible playbooks]: http://www.ansible.com/home
[Daisyproducer]: http://sbsdev.github.io/daisyproducer
[Daisyproducer2]: https://github.com/sbsdev/daisyproducer2
[Kati]: https://github.com/sbsdev/catalog
[Madras2]: https://github.com/sbsdev/mdr2


