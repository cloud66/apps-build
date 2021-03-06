FROM ubuntu:14.04

# Install required packages
RUN apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get install -qy --no-install-recommends \
      ca-certificates \
      openssh-server \
      wget \
      curl \
      apt-transport-https \
      postfix \
      nano

# Download & Install GitLab
# If you run GitLab Enterprise Edition point it to a location where you have downloaded it.
RUN echo "deb https://packages.gitlab.com/gitlab/gitlab-ce/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/gitlab_gitlab-ce.list
RUN wget -q -O - https://packages.gitlab.com/gpg.key | apt-key add -
RUN apt-get update && apt-get install -yq --no-install-recommends gitlab-ce=7.13.2-ce.0

# Manage SSHD through runit
RUN mkdir -p /opt/gitlab/sv/sshd/supervise \
    && mkfifo /opt/gitlab/sv/sshd/supervise/ok \
    && printf "#!/bin/sh\nexec 2>&1\numask 077\nexec /usr/sbin/sshd -D" > /opt/gitlab/sv/sshd/run \
    && chmod a+x /opt/gitlab/sv/sshd/run \
    && ln -s /opt/gitlab/sv/sshd /opt/gitlab/service \
    && mkdir -p /var/run/sshd

# Prepare default configuration
RUN (echo "" \
     && echo "# Cloud 66 customizations below" \
     && echo "postgresql['enable'] = false" \
     && echo "redis['enable'] = false" \
     && echo "gitlab_rails['db_adapter'] = 'postgresql'" \
     && echo "gitlab_rails['db_encoding'] = 'unicode'" \
     && echo "gitlab_rails['db_database'] = ENV[\"POSTGRESQL_DATABASE\"]" \
     && echo "gitlab_rails['db_username'] = ENV[\"POSTGRESQL_USERNAME\"]" \
     && echo "gitlab_rails['db_password'] = ENV[\"POSTGRESQL_PASSWORD\"]" \
     && echo "gitlab_rails['db_host'] = ENV[\"POSTGRESQL_ADDRESS\"]" \
     && echo "gitlab_rails['db_port'] = 5432" \
     && echo "gitlab_rails['redis_host'] = ENV[\"REDIS_ADDRESS\"]" \
     && echo "gitlab_rails['redis_port'] = 6379" \
     && echo "gitlab_rails['gitlab_shell_ssh_port'] = 10022" \
     && echo "gitlab_rails['gitlab_ssh_host'] = ENV[\"SITE\"]" \ 
     && echo "gitlab_ci['smtp_enable'] = ENV[\"SMTP_ENABLE\"] == 'true'" \ 
     && echo "gitlab_ci['smtp_address'] = ENV[\"SMTP_ADDRESS\"]" \ 
     && echo "gitlab_ci['smtp_port'] = ENV[\"SMTP_PORT\"].to_i" \ 
     && echo "gitlab_ci['smtp_user_name'] = ENV[\"SMTP_USER_NAME\"]" \ 
     && echo "gitlab_ci['smtp_password'] = ENV[\"SMTP_PASSWORD\"]" \ 
     && echo "gitlab_ci['smtp_domain'] = ENV[\"SMTP_DOMAIN\"]" \ 
     && echo "gitlab_ci['smtp_tls'] = ENV[\"SMTP_TLS\"] == 'true'") >> /etc/gitlab/gitlab.rb

# Expose web & ssh
EXPOSE 80 443 22

RUN sed -i "s/^\.*external_url.*$/external_url ENV[\"SITE\"]/" /etc/gitlab/gitlab.rb
RUN mv /etc/gitlab/gitlab.rb /tmp/gitlab.rb

# Copy assets
COPY runner.sh seeder.sh killtree.sh /usr/local/bin/
RUN chmod 700 /usr/local/bin/runner.sh /usr/local/bin/seeder.sh /usr/local/bin/killtree.sh

# Runner to handle signal, trigger runit and reconfigure GitLab
CMD ["/usr/local/bin/runner.sh"]
