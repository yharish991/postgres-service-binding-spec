[Crunchydata postgres-operator](https://github.com/CrunchyData/postgres-operator) service binding spec adoption

Multiple ways to connect to the postgres db cluster created by the crunchydata postgres operator

- Binding without TLS, using the `testuser` user credentials created when the cluster is created.

ServiceBinding CR should look like this

```
services:
  - group: ''
    kind: Secret
    name: pickles-testuser-secret
    version: v1
  - group: crunchydata.com
    kind: Pgcluster
    name: pickles
    version: v1
```

- Binding with user credentials that are created when you create the user by using command `pgo create user`

ServiceBinding CR should look like this

```
services:
  - group: ''
    kind: Secret
    name: pickles-someuser-secret
    version: v1
  - group: crunchydata.com
    kind: Pgcluster
    name: pickles
    version: v1
```


- Binding with pgbouncer that is created using command `pgo create pgbouncer`

ServiceBinding CR should look like this

```
services:
  - group: ''
    kind: Secret
    name: pickles-someuser-secret
    version: v1
  - group: crunchydata.com
    kind: Service
    name: pickles-pgbouncer
    version: v1
```

- Binding with TLS, using the testuser or any user, user credentials created when the cluster is created.(certificate generation is manual, not handled by operator)


### Annotations that need to be added in the Pgcluster CR
```
servicebinding.dev/database:"path={.spec.database},bindAs=volume"
servicebinding.dev/port:"path={.spec.port},bindAs=volume"
servicebinding.dev/host:"path={.spec},bindAs=volume,elementType=template,source={{ .spec.primaryhost }}.{{ .spec.namespace }}"
```

### Annotations that need to be added to Service created when pgbouncer is created
```
servicebinding.dev/pgbouncer_host:"path={.spec.clusterIP},bindAs=volume"
servicebinding.dev/pgbouncer_port:"path={.spec},bindAs=volume,elementType=template,source={{- range .spec.ports -}} {{- if (eq .name "postgres") -}} {{ .port }} {{- end -}} {{- end -}}"
```


Little background about Crunchydata postgres-operator
- When a new cluster(for example: pickles) is created, it creates 3 users and a service `pickles`

1.  `primaryuser` - is used for replication between primary and replicas, user credentials are stored in `pickles-primaryuser-secret`
2. `postgresuser` - is the admin user for the database instance, user credentials are stored in `pickles-postgres-secret`
3. `testuser` - is a normal user that has access to the database that is created for testing purposes, user credentials are stored in `pickles-testuser-secret`

- When a new user(for example: someuser)is created, it creates a new secret `pickles-someuser-secret` with the `someuser` user credentials

- When pgbouncer is created, it creates a secret `pickles-pgbouncer-secret` with pgbouncer user credentials and a service to connect to pgbouncer called `pickles-pgbouncer` is created
