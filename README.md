# SBS infrastructure

[Ansible playbook][] to install a [Daisyproducer][] and a Madras2 server.

## Deployment

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

### Madras2

To deploy to the madras test server run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt madras2.yml
```

## License

Copyright (c) 2014 [Swiss Library for the Blind, Visually Impaired and Print Disabled](http://www.sbs.ch/).

Licensed under the [MIT License](./LICENSE).

[Ansible playbook]: http://www.ansible.com/home
[Daisyproducer]: http://www.daisyproducer.org


