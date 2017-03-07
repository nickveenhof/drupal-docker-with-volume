FROM wodby/php:5.6

# Add Scripts
ADD scripts/init_git.sh /docker-entrypoint-init.d/init_git.sh
ADD scripts/pull /usr/bin/pull
ADD scripts/push /usr/bin/push
ADD scripts/letsencrypt-setup /usr/bin/letsencrypt-setup
ADD scripts/letsencrypt-renew /usr/bin/letsencrypt-renew
