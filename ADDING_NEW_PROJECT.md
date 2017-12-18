# Adding new project
  
To add new project follow instructions:
- Add project entry to `projects.yaml` file. Find projects orgs, repos, select start date.
- Set project databases (Influx and Postgres).
- Set it to `disabled: true` for now.
- If not using`devstats` cron job then add project entry in `crontab.entry` but do not install new cron yet (that will be the last step).
- Update `./cron/cron_db_backup_all.sh`, `reinit.sh` but do not install yet.
- Add new domain for the project: `projectname.cncftest.io`.
- Search for all files defined for some existing project, for example `find . -iname "*prometheus*"`.
- Generate icons for new project: `./grafana/img/projectname32.png`, `./grafana/img/projectname.svg`.
- And/or update `grafana/copy_artwork_icons.sh`, `apache/www/copy_icons.sh`.
- PNG should be 32bit RGBA 32x32 PNG.
- SVG should be color and square.
- Copy setup scripts and then adjust them:
- `cp -R prometheus/ projectname/`, `mv projectname/prometheus.sh projectname/projectname.sh`, `vim projectname/*`.
- You need to set correct project main GitHub repository and annotations match regexp in `projects.yaml` to have working annotations and quick ranges.
- Copy `metrics/prometheus` to `metrics/projectname`, those files will need tweaks too.
- `cp -Rv scripts/prometheus/ scripts/projectname`, `vim scripts/projectname/*`.
- Create Postgres database for new project: `sudo -u postgres psql`
- `create database projectname;`
- `grant all privileges on database "projectname" to gha_admin;`
- Generate Postgres data: `PG_PASS=... IDB_PASS=... IDB_HOST=172.17.0.1 ./grpc/grpc.sh`.
- When data is imported into Postgres: projectname, You need to update `metrics/projectname/gaps.yaml`.
- Using `./metrics/projectname/companies_tags.sql`,  `./projectname/top_n_companies.sh`.
- And `./projectname/top_n_repos_groups.sh`.
- There are two kinds of repo group names: direct from query, but also with special characters replaced with "_"
- You should copy those from query, put there where needed and do VIM replace: `:'<,'>s/[-/.,;:`\s]/_/g`.
- Run regenerate all InfluxData script `./projectname/reinit.sh`.
- `cp grafana/prometheus/* grafana/projectname/*` and then update files.
- `cp grafana/dashboards/prometheus/* grafana/dashboards/projectname/*` and then update files.
- Update: `grafana/copy_artwork_icons.sh`, `apache/www/copy_icons.sh`.
- Update `projects.yaml` remove `disabled: true` for new project.
- `make install` to install all changed stuff.
- Copy directories `/etc/grafana`, `/usr/share/grafana`, `/var/lib/grafana` froma standard unmodified installation adding .projectname to their names.
- Update `./grafana/projectname/change_icons_and_title.sh` to use right names. Run it with `GRAFANA_DATA=/usr/share/grafana.projectname/ ./grafana/projectsname/change_title_and_icons.sh`.
- Update `/etc/grafana.projectname/grafana.ini` - set all config options from `GRAFANA.md`, `MULTIPROJECT.md`.
- Start new grafana: `./grafana/projectname/grafana_start.sh`.
- Update Apache config to proxy https to new Grafana instance: `vim /etc/apache2/sites-enabled/000-default-le-ssl.conf`, `service apache2 restart`
- Issue new SSL certificate as described in `SSL.md`: `sudo certbot --apache -d 'cncftest.io,k8s.cncftest.io,prometheus.cncftest.io,opentracing.cncftest.io,fluentd.cncftest.io,linkerd.cncftest.io,newproject.cncftest.io'`
- Or `sudo certbot --apache -d 'devstats.cncf.io,k8s.devstats.cncf.io,prometheus.devstats.cncf.io,opentracing.devstats.cncf.io,fluentd.devstats.cncf.io,linkerd.devstats.cncf.io,newproject.devstats.cncf.io'`
- Open `newproject.cncftest.io` login with admin/admin, change the default password! and follow instructions from `GRAFANA.md`.
- Add new project to `/var/www/html/index.html`.