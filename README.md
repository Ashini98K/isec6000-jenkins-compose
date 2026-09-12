## ISEC6000 Assessment 2

Forked from aws-samples/aws-elastic-beanstalk-express-js-sample.

This repository holds the **application**: source, unit tests, Dockerfile and
the Jenkinsfile that builds it. The Jenkins environment that runs the pipeline
is defined separately in https://github.com/Ashini98K/isec6000-jenkins-compose.

The split is deliberate — application code changes many times a day and is
built by anyone with commit access, while the CI configuration defines *who*
may run those builds and with what privileges. Different change rate,
different review requirements.
