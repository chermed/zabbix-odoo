
# Template Odoo for Zabbix

## Overview

Monitor load time of requests, errors, warnings, etc based on logfile and ping if the server is up or down
Some triggers are included to use them in actions (sending emails)
A graph of loading time is provided in a screen.
This template was tested on Zabbix 4.2.0 and Odoo vesion 12 on Linux.

## Setup

1. Add to Odoo log handler: `odoo.http.rpc.request:DEBUG` or start Odoo with the parameter `--log-request` 
2. Import `template_odoo.xml` to Zabbix and link it to the target host
3. Disable not needed items and triggers

## Zabbix configuration

No specific Zabbix configuration is required.

### Macros used

| Macro                  | Description                                   | Default                 |
|------------------------|-----------------------------------------------|-------------------------|
| {$ODOO.LOGFILE}        | Path of odoo log                              | /var/log/odoo/odoo.log  |
| {$ODOO.HOST}           | Host or domain on which Odoo is available     | 127.0.0.1               |
| {$ODOO.PORT}           | Port of which Odoo is exposed                 | 8069                    |
| {$ODOO.PATH.TEST}      | Path to test if it return code HTTP 200       | /web/login              |
| {$ODOO.LOAD.MAX}       | Maximum value in seconds to consider as slow  | 2                       |
| {$ODOO.ENV}            | The name of the environment                   | main                    |

## Template links

There are no template links in this template.

## Discovery rules

There are no discovery rules

## Items collected

	
| Name                                       | Description         | Type           |
|--------------------------------------------|---------------------|----------------|
| Odoo [{$ODOO.ENV}]: All logs               | Get all lines of the logfile  | Zabbix agent active  |
| Odoo [{$ODOO.ENV}]: Access Denied by ACLs  | Lines of access denied by ACLS  | Dependant item  |
| Odoo [{$ODOO.ENV}]: Errors                 | All error lines   | Dependant item   |
| Odoo [{$ODOO.ENV}]: Errors with Traceback  | Get all error lines  | Dependant item   |
| Odoo [{$ODOO.ENV}]: Warnings               | Get all warning lines and traceback  | Dependant item   |
| Odoo [{$ODOO.ENV}]: Login failed           | Lines of failed login  | Dependant item   |
| Odoo [{$ODOO.ENV}]: Operation Denied (Rules) | Lines of all denied operations by rules  | Dependant item   |
| Odoo [{$ODOO.ENV}]: Load time            | Get the load time (elapsed time to all api calls | Dependant item   |
| Odoo [{$ODOO.ENV}]: Server availability    | Ping the host and port if it listen | Zabbix agent active   |
| Odoo [{$ODOO.ENV}]: status code            | Get the {$ODOO.PATH.TEST} response and extract the status code | Zabbix agent active   |


## Triggers

| Name                                       | Severity    | Expression                                           |       
|--------------------------------------------|-------------|------------------------------------------------------|
| Odoo [{$ODOO.ENV}]: Access Denied by ACLs  | Information | {Template Odoo:odoo.error.access.denied.last(#1)}<>0 | 
| Odoo [{$ODOO.ENV}]: Error                  | High        | 	{Template Odoo:odoo.error.nodata(60s)}=0            |
| Odoo [{$ODOO.ENV}]: Load over {$ODOO.LOAD.MAX} seconds |High| {Template Odoo:odoo.load.time.last(#1)}>{$ODOO.LOAD.MAX} |
| Odoo [{$ODOO.ENV}]: Server is down |Disaster|{Template Odoo:net.tcp.service[http,{$ODOO.HOST},{$ODOO.PORT}].last(#1)}=0|
| Odoo [{$ODOO.ENV}]: Status code <> 200|Disaster|	{Template Odoo:web.page.get[{$ODOO.HOST},{$ODOO.PATH.TEST},{$ODOO.PORT}].last(#1)}<>200|
| Odoo [{$ODOO.ENV}]: Warning                | Warning     | {Template Odoo:odoo.warning.nodata(60s)}=0            |
