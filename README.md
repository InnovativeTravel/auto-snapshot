humilis-autosnapshot
=============

[![Build Status](https://api.travis-ci.org/InnovativeTravel/auto-snapshot.svg)](https://travis-ci.org/InnovativeTravel/auto-snapshot/)

Creates scheduled snapshots of selected EBS volumes in the AWS cloud. 

__NOTE:__ This repo has been deprecated by the more comprehensive
[EBS snapshot scheduler][ebsscheduler] supported by AWS.

[ebsscheduler]: https://aws.amazon.com/answers/infrastructure-management/ebs-snapshot-scheduler/

# How does it work?

Once deployed in the AWS cloud, the system creates snapshots from every EBS
volume that has the following tags:

```
tag key                             tag value
-----------------------------------------------------------------------------------
*Name                               <volume friendly name>
*auto:snapshots                     <any value>
auto:snapshot:topic                 <topic name for SNS event>
auto:snapshots:retention_days       <number of days to keep the snapshot>
```
*mandatory tags


## Example tags:

```
tag key                             tag value
-----------------------------------------------------------------------------------
Name                                my-super-important-volume
auto:snapshots                      yes
auto:snapshot:topic                 snapshot-was-created-for-my-super-important-volume
auto:snapshots:retention_days       15
```


# Development

Assuming you have [virtualenv][venv] installed:

[venv]: https://virtualenv.readthedocs.org/en/latest/

```
cd lambda
make develop

. .env/bin/activate
```

To run the test suite:

```
make test
```


# Deployment

__Note__: if you don't know what [humilis][humilis] it, it is strongly
recommended that you take a quick look at the [docs][humilis] before reading
further.

First you will need to install and configure [humilis][humilis], if you 
haven't do so already. Then you can make a test deployment to AWS using:


```bash
make create
```

which will deploy in stage `TEST` a dummy environment that contains just the
`autosnapshot` layer. You can then go to the AWS Lambda console and test the
deployment there.

In a real deployment the `autosnapshot` layer is unlikely to stand on its own,
but to be part of a larger environment. Your environment definition file may
look something like this:

```yaml
my-environment:
    description:
        Everything I need to keep my company running
    layers:
        # The vpc layer takes care of setting the networking stuff
        - {layer: vpc}
        # Deploy a layer with some RDS instances
        - {layer: databases}
        # Deploy a layer with some EC2 instances
        - {layer: instances}
        # Deploy the autosnapshot layer
        - {layer: authosnapshot}
```

This means that your environment directory tree may look something like this:


```
.
├── layers
│   ├── autosnapshots
│   │   ├── meta.yaml
│   │   ├── outputs.json.j2
│   │   └── resources.json.j2
│   ├── dababases
│   ├── instances
│   └── vpc
└── myenvironment.yaml
```

where the `layers/autosnapshots` directory has the same contents as this
repository. The best approach to do this is to add this repository as a
[subtree][git-subtrees] of your environment repository. I recommend you use
[stree][stree] to manage git subtrees:

[humilis]: https://github.com/InnovativeTravel/humilis/blob/master/README.md
[git-subtrees]: https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec#.8w3u599dv
[stree]: https://github.com/deliciousinsights/git-stree

```
# Assume you are in the root directory of your environment, i.e. where the
# environment definition file is located.
git stree add autosnapshot -P layers/autosnapshot \
    git://github.com/Innovativetravel/auto-snapshot.git
```


Once the autosnapshot subtree has been added to your environment repository, 
you would deploy the snapshot layer as part of the deployment of your whole
environment. E.g. by running a command similar to this one:

```
humilis --profile [profile] create --stage [stage] myenvironment.yaml
```

where `[profile]` is the [humilis][humilis] configuration profile (which
determines, e.g. which AWS credentials should be used for deployment), and
`[stage]` may be used when you want to have multiple deployments of your
environment running side by side (e.g. by deploying first to stage `DEV` and
then to stage `PROD`).

