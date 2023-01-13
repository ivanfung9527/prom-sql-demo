# prom-sql-demo

## About
This repo demostrates how to setup prometheus to monitor RDBMS tables through query result, using Docker and Docker Compose. Email alert is sent to destinated address (in `alertmanager.yml`) by Prometheus based on pre-defined rules in `rules/rule1.rules`. The following tools are used (in `docker-compose.yml`):

* Prometheus
* SQLagent
* prometheus-sql
* prom-alertmanager
* MySQL
* phpmyadmin

## Environment
* Unbuntu-20.04 LTS
* Docker-20.10.22
* Docker Compose version v2.14.1

## Quickstart
1. Update the email sender and receiver details in `alertmanager.yml`
```yaml
global:
  smtp_smarthost: 'smtp-mail.outlook.com:587'
  smtp_from: 'test-outlook@outlook.com'
  smtp_auth_username: 'test-outlook@outlook.com'
  smtp_auth_identity: 'test-outlook@outlook.com'
  smtp_auth_password: 'test-outlook-app-pw'

route:
  receiver: 'my-gmail'

  group_by: ['Response_Time_Count_Exceed_5', 'Nr_Companies_per_Country_SWE_Exceed_3']

  group_wait: 10s

  group_interval: 10s

  repeat_interval: 3h

receivers:
- name: 'my-gmail'
  email_configs:
  - to: 'test-gmail@gmail.com'
```

2. Start the containers
```bash
docker compose up
```
3. Go to phpmyadmin at `localhost:8081`. The username and password is `test` and `unsecure` as defined in `config.yml`

4. Run the following queries to initiate tables.
```SQL
INSERT INTO Companies VALUES
(5, 'Company3', 'SWE'),
(6, 'Company4', 'SWE'),
(7, 'Company5', 'SWE'),
(8, 'Company6', 'SWE');

INSERT INTO Requests VALUES
(5, 100),
(6, 37);
```
5. The receiver should get an email alert after some time.

## Customization
### 1. Receivers and alerts
There can be multiple receivers and alerts defined in `alertmanager.yml`. Each of the receivers can subscribe different set of alerts. Refer to [offical alertmanger Github](https://github.com/prometheus/alertmanager) for detailed guide.

### 2. Rules and queries
The queries and corresponding rules are defined in `queries.yml` and `rules/rule1.rules`.

`queries.yml`
```yaml
- nr_companies_per_country:
    sql: >
        select country, count(1) as cnt from Companies group by country
    data-field: cnt
    # inertval: query interval in secound
```
`rule1.rules`
```yaml
groups:
  - name: example-rule2
    rules:
      - alert: Nr_Companies_per_Country_SWE_Exceed_3
        expr: query_result_nr_companies_per_country{country="swe"} > 3
        for: 10s
        labels:
          severity: warning
        annotations:
          # Alert message
          summary: "Number of company in SWE exceed 3, currently {{$value}}"
```


## Reference
* [Monitoring Data in a SQL Table with Prometheus and Grafana](https://jcooney.net/post/2017/10/23/prometheus-grafana-sql.html)
* [Prometheus SQL](https://github.com/chop-dbhi/prometheus-sql)