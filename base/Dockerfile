# docker-gitlab-ci-runner

FROM gitlabhq/gitlab-ci-runner
MAINTAINER  James Dinsdale "hi@molovo.co"

ADD files/run /sbin/run
RUN chmod +x /sbin/run

VOLUME /root/.ssh

CMD "/sbin/run"
