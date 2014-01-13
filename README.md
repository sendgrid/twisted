twisted
=======

## Create twisted RPM

OPTIONAL: set next build number (defaults to 0)

Updates that apply to all sendgrid twisted versions should be applied
to the sendgrid_package_patch branch and merged into each supported
version branch.  Separate patch branches may be created as needed.

## Currently Supported

Sendgrid currently has two builds of twisted, each build by Jenkins

* 10.0.0 http://build.sendgrid.net:8080/job/twisted/
* 13.2.0 http://build.sendgrid.net:8080/job/twisted-13/

## Create new sendgird version
Example create 13.3.0
* assume origin git@github.com:sendgird/twisted
* assume all remotes and branches are up to date

1. Check out twisted tag of version 13.3.0
```shell
$ git checkout twisted-13.3.0
```

2. Create branch sendgrid-13.3.0 from twisted tag twisted-13.3.0
```shell
$ git co -b sendgrid-13.3.0
```

3. Rebase sendgrid_package_patch branch onto sendgrid-12.0.0
```shell
$ git rebase sendgrid_package_patch
```

4. Push branch to origin and create jenkins job to build rpm

## Create twisted RPM 13.3.0-0
1. Checkout sendgrid version branch
```shell
$ git checkout sendgird-13.3.0
```

2. OPTIONAL: set next build number (defaults to 0)
```shell
$ export BUILD_NUMBER=$YOUR_NEXT_BUILD_NUMBER
```

3. In the repo root directory run:

```shell
$ bash -le .jenkins
```
