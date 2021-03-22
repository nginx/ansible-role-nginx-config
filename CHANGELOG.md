# Changelog

## 0.4.0 (Unreleased)

BREAKING CHANGES:

*   Rename upload related variables:
    *   Rename the `nginx_config_snippet_upload_*` parameters to `nginx_config_upload_*` (check `defaults/main/upload.yml` for an example).
    *   Rename the `nginx_config_html_upload_*` parameters to `nginx_config_upload_html_*`.
    *   Rename the `nginx_config_ssl_upload_*` parameters to `nginx_config_upload_ssl_*`.
*   Tweak the `nginx_config_html_upload` and `nginx_config_ssl_upload` parameters to ow use a list instead of a single `src` and `dest` value (check `defaults/main/upload.yml` for an example).
*   Refactor the base config templates to simplify the creation of templates as well as development and maintenance moving forward:
    *   Modify `servers`, `servers.listen`, `server.locations`, `upstream` and `upstream.servers` from nested dictionaries in the `http` and `stream` configuration templates to lists, as well as modify the `nginx_config_html_demo_template` variable from a nested dictionary to a list. To update your templates, replace the aforementioned nested dictionary keys by lists (place a dash in front of the topmost nested value within each aforementioned nested dictionary).
    *   Remove/merge the `web_server` and `reverse_proxy` nested dictionary keys from the HTTP templates. These often lead to confusing and unnecessary code duplication and hard to maintain code. To update your templates, remove both keys and adjust your spacing accordingly.
*   Refactor the `upstream` HTTP config template into its own separate file. The following variables have changed (check [`defaults/main/template.yml`](https://github.com/nginxinc/ansible-role-nginx-config/blob/main/defaults/main/template.yml) for examples):
    *   `port` is no longer supported. Instead, include the port as part of your `address`.
    *   `lb_method` is no longer supported. Instead, you will have to specifically set the method you want to use.
    *   `zone_name` and `zone_size` have been modified into a dictionary.
    *   `sticky_cookie` is no longer supported as is. You will now have to configure the `sticky_cookie` values.
    *   The `health_check` parameter within the `server` dictionary is no longer supported. Instead, manually set `max_fails` and `fail_timeout`.
*   Refactor the `proxy` HTTP config template into its own separate file. All variables have changed (check [`defaults/main/template.yml`](https://github.com/nginxinc/ansible-role-nginx-config/blob/main/defaults/main/template.yml) for examples):
    *   All `proxy_*` related variables now live under the `proxy` dictionary key, with the exception of `proxy_pass` and `proxy_cache_path`. You can specify the `proxy` dictionary key inside the `http`, `server`, and `location` contexts.
    *   Remove the `nginx_config_main_template.http_settings.cache` dictionary variable. Use `nginx_config_http_template.*.proxy_cache_path` instead.
    *   Remove the `location.websocket` variable. Use `location.proxy.set_header` instead:
    ```yaml
    proxy:
      set_header:
        - field: Upgrade
          value: $http_upgrade
        - field: Connection
          value: Upgrade
    ```
*   Rename some NGINX template config parameters to align with NGINX directive names:
    *   Rename `proxy_hide_headers` to `proxy_hide_header`.
    *   Rename `html_file_location` to `root`.
    *   Rename `html_file_name` to `index`.

DEPRECATION WARNINGS:

The `nginx_config_main_upload_*`, `nginx_config_upload_html_*`, and `nginx_config_stream_upload_*` parameters have been deprecated in favor of a newly introduced parameter, `nginx_config_upload_*` (previously `nginx_config_snippet_upload_*`). The new parameter provides greater flexibility in configuring your upload settings in addition to simplifying the upload Ansible tasks. The deprecated parameters will be removed in the next major release (0.5.0), due April 2021.

FEATURES:

Replace Ansible community distribution with Ansible base and add the necessary extra collections as a dependency requirement. For reference, these are:
*   `community.general`
*   `ansible.posix`

ENHANCEMENTS:

*   Add support for NGINX's `index` directive to the `server` block within the template config parameters.
*   Update Ansible base to `2.10.7`, Molecule to `3.2.4`, yamllint to `1.26.0` and Docker Python SDK to `4.4.4`.
*   Specify GitHub actions Ubuntu release.
*   Minor GitHub template tweaks, including the creation of a SECURITY doc.

BUG FIXES:

*   Add `state` parameter to package module in Molecule verification tests.
*   In App Protect environments on SELinux enforced systems, the `nginx -t` handler fails when run from a directory that the nginx process' user does not have access to.
*   Fix missing GRPC boolean check in GRPC template.

## 0.3.3 (January 28, 2021)

FEATURES:

*   Add support for configuration snippets (use the `nginx_config_snippet_upload_*` parameters to configure it).
*   Add support for Dependabot.

ENHANCEMENTS:

*   Add support for NGINX GRPC directives.
*   Add support for NGINX GZIP directives.
*   Add support for upstream server `backup` parameter in http and stream template.
*   Only run GitHub actions Galaxy CI/CD workflow when a new release is published.
*   Update list of supported platforms.
*   Update Ansible base to `2.10.5` and Ansible to `2.10.6`.

BUG FIXES:

*   Address inconsistent types within Jinja templates.
*   Add Jinja2 checks to all config template parameters to ensure that they are only included when appropriately defined.
*   Fix edge case where `proxy_pass` is still required when using `grpc_pass`.

## 0.3.2 (January 11, 2021)

ENHANCEMENTS:

*   The GitHub actions Molecule CI/CD workflow is no longer run on a new release (this is not necessary since it already runs on every push).
*   The GitHub actions Molecule CI/CD workflow should now correctly avoid running 'Plus' related tests on external PRs.
*   The `cleanup-config.yml` playbook has been slightly refactored and simplified.
*   Update Ansible base to `2.10.4`, Ansible to `2.10.5`, Molecule to `3.2.2` and Docker Python SDK to `4.4.1`.
*   Update copyright notice.

## 0.3.1 (December 22, 2020)

ENHANCEMENTS:

*   Update Molecule to `3.2.1` and Docker Python SDK to `4.4.0`.
*   Replace TravisCI with GitHub actions.

BUG FIXES:

*   Switch to explicit boolean values in `sub_filter` defaults for `last_modified` and `since` in `nginx_config_main_template`. `"on"` and `"off"` values are treated as true instead of true/false when surrounded by double quotes. By always resorting to true/false we avoid unaccounted edge cases.
*   Fix issue whereas SELinux state would not be correctly set back to `enforcing` when `nginx_config_selinux: true`.

## 0.3.0 (November 17, 2020)

BREAKING CHANGES:

*   The default port of the status and REST API config is now `8080` and matches the CI Molecule test which already uses it. You can set `nginx_config_status_port` to another value if desired.
*   The allow/deny directives for `nginx_config_status` and `nginx_config_rest_api` now take a list instead of a single value.
*   The default `nginx_config_*_log` values have changed to `nginx_config_*_access_log` and no longer have a default value of `off`. Set the respective variables to `false` to preserve the previous behaviour.

ENHANCEMENTS:

*   Add survey to README.
*   Improve README structure and use tables where relevant.
*   Update Ansible (now Ansible base) to `2.10.3`, Ansible (now Ansible Community Distribution) to `2.10.3`, Ansible Lint to `4.3.7`, Molecule to `3.1.5`, and yamllint to `1.25.0`.
*   Improve templating of stub status and REST API config.

BUG FIXES:

*   Prevent TravisCI from trying to build (and failing) NGINX Plus images on external PRs.
*   Fix naming for SELinux facts dictionary.
*   Correctly import `app_protect` global directives in template.
*   Role now runs correctly when using Ansible's check mode.
*   Fix issue with access log in stub status and REST API config template not being properly parsed.

## 0.2.0 (September 24, 2020)

BREAKING CHANGES:

The process to configure modules has changed. Instead of manually setting the modules you want to install to `true` or `false`, you will now have to use either:
*   A newly introduced top level list variable, `nginx_config_modules`.
*   A newly introduced list variable within your main NGINX config template, `nginx_config_main_template.modules`.

Make sure you only use one variable or the other, since they will overwrite each other. This change will simplify adding future supported modules to this role, and allows you to include any external modules you may wish in your NGINX config.

FEATURES:

*   Support for all NGINX App Protect directives has been added. You can find details on the supported directives on `defaults/main/template.yml`. This is the first module to be included using J2 macros. Expect to slowly see a refactor of various modules to use macros where possible.
*   Add Alpine `3.12` to the list of supported platforms.
*   Remove Alpine `3.8` from the list of supported platforms .
*   Add NGINX Plus tests to TravisCI

ENHANCEMENTS:

*   Added handlers to check for NGINX syntax validity and fail if any errors are detected.
*   Switch to using `ansible_facts` wherever possible.
*   Improved tasks naming conventions.
*   Update Ansible to `2.9.13` and Ansible Lint to `4.3.5`.
*   Explicitly defined `mode` in relevant tasks.
*   Improve configuration templating capabilities:
    *   Allow setting `access_log`/`access_log_location` to `off`.
    *   Add IP restriction for web servers

BUG FIXES:

An empty `nginx_config_cleanup_files` will no longer cause `nginx_config_cleanup` related tasks to fail.

## 0.1.0 (August 19, 2020)

Initial release of the NGINX Config role. Contains all NGINX Config related features previously available on the [NGINX Ansible role](https://github.com/nginxinc/ansible-role-nginx).
