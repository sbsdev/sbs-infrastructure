# SBS infrastructure

[Ansible playbook][] to install a [Daisyproducer][] server.

## Deployment

Put the needed debs into `files`. Adapt the version numbers in
`host_vars/repo` (adapt the two variables `staging_debs` and
`production_debs`). To add the debs to the repo run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt apt-repo.yml
```

To actually deploy to the test server run

```
ansible-playbook -i test -K --vault-password-file .vault_pass.txt daisyproducer.yml
```

To deploy to the production server run

```
ansible-playbook -i production -K --vault-password-file .vault_pass.txt daisyproducer.yml
```

## License

Copyright 2014 Swiss Library for the Blind, Visually Impaired and Print Disabled.

Licensed under the [MIT License](./LICENSE).

[Ansible playbook]: http://www.ansible.com/home
[Daisyproducer]: http://www.daisyproducer.org


