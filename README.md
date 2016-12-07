Role Name
=========

Ansible Role to Install and Configure Logstash

[![Build Status](https://travis-ci.org/valentinogagliardi/logstash-role.svg?branch=master)](https://travis-ci.org/valentinogagliardi/logstash-role)

Requirements
------------

**Java** should be present on the nodes machines in order to run Logstash. This role does not install Java.

On Debian OS family, **python-pycurl** and **python-apt** are required to deal with apt Ansible modules. The role already take care of these dependencies.

Example Playbooks
----------------

```yaml
- hosts: LogstashNodes
  roles:
    - role: valentinogagliardi.logstash-role

      logstash_defaults: |
        LS_HEAP_SIZE="256m"
        LS_OPTS="-r --config.reload.interval 2"

      logstash_patterns: |
        TIMESTAMP_FOO %{MONTHDAY}-%{MONTH}-%{YEAR}-%{HOUR}:%{MINUTE}:%{SECOND}

      logstash_inputs: |
        syslog { host => "{{ ansible_eth0.ipv4.address }}"
                port => "514"
                type => "syslog_input"
              }

        syslog { host => "{{ ansible_lo.ipv4.address }}"
                port => "515"
                type => "syslog_input_local"
              }

        grok { patterns_dir => [ "./patterns" ]
              match => { "message" => "%{TIMESTAMP_FOO:[@metadata][timestamp]} %{GREEDYDATA:message}" }
            }

      logstash_filters: |
        geoip { source => "ip_address"
             }

        multiline { pattern => "^No lfn2pfn"
                   what => "previous"
                 }

      logstash_outputs: |
        file { path => "/var/log/logstash/output.log"
            }
```

Role Variables
--------------

```yaml
# Increase Java performace
# http://stackoverflow.com/questions/137212/how-to-solve-performance-problem-with-java-securerandom
logstash_java_opts: "-Djava.security.egd=file:/dev/./urandom"

logstash_python_utils:
 - { package: "python-pycurl" }
 - { package: "python-apt" }

# Custom vars for backward compatibility
logstash_custom_vars:
    - {
        version: ["5.0"],
        apt_repo: "deb https://artifacts.elastic.co/packages/5.x/apt stable main",
        repo_key: "https://artifacts.elastic.co/GPG-KEY-elasticsearch",
        new_way_configure: "True",
        home_dir: "/usr/share/logstash",
        plugin_bin: "logstash-plugin",
        config_validate_params: "--path.settings {{ logstash_settings }} -tf %s"
      }
    - {
        version: ["1.5", "2.0", "2.1", "2.2", "2.3", "2.4"],
        apt_repo: "deb http://packages.elasticsearch.org/logstash/{{ logstash_version }}/debian stable main",
        repo_key: "http://packages.elasticsearch.org/GPG-KEY-elasticsearch",
        new_way_configure: "False",
        home_dir: "/opt/logstash",
        plugin_bin: "plugin",
        config_validate_params: "-tf %s"
      }

# Default vars, if we set the wrong version logstash.
logstash_version: "2.4"
logstash_apt_repo: "deb http://packages.elasticsearch.org/logstash/{{ logstash_version }}/debian stable main"
logstash_repo_key: "http://packages.elasticsearch.org/GPG-KEY-elasticsearch"
logstash_new_way_configure: "False"
logstash_home: "/opt/logstash"
logstash_plugin_bin: "plugin"
logstash_config_validate_params: "-tf %s"

logstash_yum_repo_dest: "/etc/yum.repos.d/logstash.repo"

logstash_conf_dir: "/etc/logstash/conf.d/"
logstash_settings: "/etc/logstash"
logstash_patterns_file: "/etc/logstash/patterns/extra"

defaults_RedHat: "/etc/sysconfig/logstash"
defaults_Debian: "/etc/default/logstash"

# following options are required so logstash can be setup correctly as a 'service'
logstash_startup_options_service_name: "logstash"
logstash_startup_options_service_description: "logstash"
logstash_startup_options_user: "logstash"
logstash_startup_options_group: "logstash"
logstash_startup_options_home: "{{ logstash_home }}"
logstash_startup_options_settings_dir: "{{ logstash_settings }}"
logstash_startup_options_opts: ""
logstash_startup_options_nice: 19
logstash_startup_options_open_files: 16384
logstash_startup_options_prestart: ""

logstash_plugins:
  - { plugin_name: "logstash-output-null" }
```

Some words
----------

Available Logstash versions: 1.5, 2.0, 2.1, 2.2, 2.3, 2.4, 5.0

The default is Logstash 2.4.

If the wrong version Logstash will be set - an incorrect repository for Logstash will be added, and the installation will not work.

Because in Logstash 5.0 have been major changes in paths, repositories and other things, we had to introduce the variable **logstash_custom_vars**, which contains specific variables for each version Logstash. This allows us to maintain backward compatibility in the role and easily be updated to version 5.0 (see the current builds status).

License
-------

GNU General Public License Version 2

Author Information
------------------

Valentino Gagliardi - valentino.g@servermanaged.it
