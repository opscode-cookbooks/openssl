openssl Cookbook
================
This cookbook provides a library method to generate secure random passwords in recipes using the Ruby OpenSSL library.

It also provides an attribute-driven recipe for upgrading OpenSSL packages.


Requirements
------------
The upgrade recipe works on the following tested platforms:

* Ubuntu 12.04, 14.04
* Debian 7.4
* CentOS 6.5

It may work on other platforms or versions of the above platforms with or without modification.

[Chef Sugar](https://github.com/sethvargo/chef-sugar) was introduced as a dependency to provide helpers that make the default attribute settings (see Attributes) easier to reason about.


Attributes
----------
* `node['openssl']['packages']` - An array of packages of openssl. The default attributes attempt to be smart about which packages are the default, but this may need to be changed by users of the `openssl::upgrade` recipe.
* `node['openssl']['restart_services']` - An array of service resources that use the `node['openssl']['packages']`. This is empty by default as Chef has no reliably reasonable way to detect which applications or services are compiled against these packages. *Note* These each need to be "`service`" resources specified somewhere in the recipes in the node's run list.


Recipes
-------
### upgrade

The upgrade recipe iterates over the list of packages in the `node['openssl']['packages']` attribute and manages them with the `:upgrade` action. Each package will send `:restart` notification to service resources named by the `node['openssl']['restart_services']` attribute.

To use the `openssl::upgrade` recipe, set the attributes as mentioned above. For example, we have a "stats_collector" service that uses openssl. It has a recipe that looks like this:

```ruby
node.default['openssl']['restart_services'] = ['stats_collector']

# other recipe code here...
service 'stats_collector' do
  action [:enable, :start]
end

include_recipe 'openssl::upgrade'
```

This will ensure that openssl is upgraded to the latest version so the `stats_collector` service won't be exploited (hopefully!).


Mixins
------
There are two mixins packaged with this cookbook.

### `OpenSSLCookbook::RandomPassword`
The `RandomPassword` mixin can be used to generate random passwords secure in Chef cookbooks. It uses Ruby's SecureRandom library and is customizable.

```ruby
Chef::Recipe.send(:include, OpenSSLCookbook::RandomPassword)
node.set["my_secure_attribute"] = random_password
node.set["my_secure_attribute"] = random_password(length: 50)
node.set["my_secure_attribute"] = random_password(length: 50, mode: :base64)
node.set["my_secure_attribute"] = random_password(length: 50, mode: :base64, encoding: "ASCII")
```

For a full list of methods and options, please consult the API documentation in the source code (`libraries/random_password.rb`).

### `Opscode::OpenSSL::Password`
This library should be considered deprecated and may be removed in a future version. Please use `OpenSSLCookbook::RandomPassword` instead. The documentation is kept here for historical reasons.

```ruby
Chef::Recipe.send(:include, ::Opscode::OpenSSL::Password)
node.set["my_secure_attribute"] = secure_password
```


LWRP
----
This cookbook includes an LWRP for generating Self Signed Certificates

### openssl_x509
generate a pem formatted x509 cert + key

#### Attributes
`common_name` A String representing the `CN` ssl field
`org` A String representing the `O` ssl field
`org_unit` A String representing the `OU` ssl field
`country` A String representing the `C` ssl field
`expire` A Fixnum reprenting the number of days from _now_ to expire the cert
`key_file` Optional A string to the key file to use. If no key is present it will generate and store one.
`key_pass` A String that is the key's passphrase
`key_length` A Fixnum reprenting your desired Bit Length _Default: 2048_
`owner` The owner of the files _Default: "root"_
`group` The group of the files _Default: "root"_
`mode`  The mode to store the files in _Default: "0400"_

#### Example usage

```ruby
openssl_x509 "/tmp/mycert.pem" do
  common_name "www.f00bar.com"
  org "Foo Bar"
  org_unit "Lab"
  country "US"
end
```


License & Authors
-----------------
- Jesse Nelson (<spheromak@gmail.com>)
- Joshua Timberman (<joshua@opscode.com>)
- Seth Vargo (<sethvargo@gmail.com>)

```text
Copyright:: 2009-2011, Opscode, Inc
Copyright:: 2014-2015, Chef Software, Inc <legal@getchef.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
