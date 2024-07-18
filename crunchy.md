# <p style="text-align: center; font-family: 'Times New Roman', serif;">
  <font size="29.5"><ins>Crunchy-Postgres-Exporter</ins></font>
</p>

</br>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="14">1. Task requirement:</font>
</p>

* <p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="4"> To set up Postgres exporter which capture the slow query also

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="14">2. Environment details:</font>

* <p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="4">Podman - Version 3 . </font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="14">3. List of tools and technologies: </font></p>

* <p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="4"> Podman Version-12 </font></p>

* <p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="4"> Postgres version -12 </font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="10">a. Definition of Podman</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="4">Podman (the POD manager) is an open source tool for developing, managing, and running containers on your Linux systems. Originally developed by Red Hat® engineers along with the open source community, Podman manages the entire container ecosystem using the libpod library.</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="10">b. Definition of PostgreSQL</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="4">PostgreSQL, also known as Postgres, is a free and open-source relational database management system emphasizing extensibility and SQL compliance. It was originally named POSTGRES, referring to its origins as a successor to the Ingres database developed at the University of California, Berkeley.</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="19.5">4. Command for the setup or configuration: </font></p>

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">1. Create Pod: </font></p>** 

> podman pod create --name crunchy-postgres --publish 9090:9090 --publish 9187:9187 --publish 5432:5432 --publish 3000:3000 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">2. Create Postgres container:</font></p>** 


> podman run -d --pod crunchy-postgres --name postgres_crunchy -e "POSTGRES_DB=postgres" -e "POSTGRES_USER=postgres" -e "POSTGRES_PASSWORD=redhat" -v 
/home/yogendra/shiksha_portal/crunchy/postgres/data:/var/lib/postgresql/data docker.io/postgres:12 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">3. Do Changes in configuration file:</font></p>**

>echo "shared_preload_libraries = 'pg_stat_statements,auto_explain'" >> postgresql.conf 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
<font size="5">4. Pull crunchy-postgres-exporter image:</font></p>** 

>podman pull 
registry.connect.redhat.com/crunchydata/crunchy-postgres-exporter:latest 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">5. Create a demo container of crunchy for setup.sql.</font></p>** 

>podman run -itd --pod crunchy-postgres --name crunchy -e 
EXPORTER_PG_PASSWORD=redhat 615904c619c5 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">6. Get the setup.sql from /opt/cpm/conf/pgxx/setup.sql from the crunchy container according to your postgres version.</font></p>**

> podman cp crunchy:/opt/cpm/conf/pg12/setup.sql .

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">7. Remove the test container of crunchy:</font></p>** 

> podman rm -f crunchy 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">8. Push setup.sql in postgres database:</font></p>** 

Get the setup.sql from /opt/cpm/conf/pg12/setup.sql from the exporter container.

> psql -h 127.0.0.1 -U postgres -d template1 < setup.sql 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">9. Create Extension:</font></p>** 

>psql -h 127.0.0.1 -U postgres -d template1 -c "CREATE EXTENSION pg_stat_statements;"

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">10. Create password for user ccp_monitoring:</font></p>**

> postgres=# \password ccp_monitoring 

> Enter new password for user "ccp_monitoring": 

**Enter it again:**

> postgres=# create database yogendra; 

> CREATE DATABASE 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">11. Now create crunchy-postgres-exporter container:</font></p>**

> podman run -itd --pod crunchy-postgres --name crunchy -e 
EXPORTER_PG_PASSWORD=redhat -e EXPORTER_PG_HOST=127.0.0.1 -e EXPORTER_PG_USER=ccp_monitoring -e 
DATA_SOURCE_NAME=postgresql://ccp_monitoring:redhat@127.0.0.1:5432/yo gendra?sslmode=disable 615904c619c5 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">12. Check metrics:</font></p>** 

> curl localhost:9187/metrics | grep query 

**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">13. Create prometheus container:<font></p>**

  </br>

> <p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">podman run -itd --pod crunchy-postgres --name prometheus_crunchy -v /home/yogendra/shiksha_portal/prometheus_crunchy/prometheus.yml:/etc/pro metheus/prometheus.yml docker.io/prom/prometheus</font></p>



> Set the target in prometheus.yml file to get metrics in prometheus and hit on browser: http://localhost:9091/


**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">14. Create Grafana Container:</font></p>** 

> podman run -itd --pod crunchy-postgres --name grafana_crunchy 
docker.io/grafana/grafana 

Hit on browser: 

> http://localhost:3000/ 
Select the prometheus as a datasource and import the dashboard.
9628

 

**Test cases list**
<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5"> - SNO </font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5"> - Component/Tool Name</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5"> - Test case</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">- Test Count</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">- Test Cases</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
<font size="5"> - Expected Result</font></p>

 <p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5"> - Test Passed [PASS/FAIL]</font></p>

<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5"> - Remarks</font></p>



<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">Note : NA</font></p>


**<p style="text-align: left; font-family: 'Times New Roman', serif;">
  <font size="5">Reference link</font></p>**

https://access.crunchydata.com/documentation/pgmonitor/2.2/exporter/index.html 

