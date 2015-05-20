# puppet-hound
Puppet module for [Hound](https://github.com/etsy/Hound)

#### Table of Contents

1. [Getting started](#getting-started)
2. [Proxy with Nginx](#proxy-with-nginx)
3. [Adding repos](#adding-repos)
4. [Fetch hound package](#fetch-hound-package)
5. [Managed vs Unmanaged](#managed-vs-unmanaged)
6. [Limitations](#limitations)
7. [TODO](#todo)
8. [Development](#development)


## Getting started
```
  class { '::hound':
    version     => '0.2.0',
    package_url => 'http://some.fqdn.tld/package.tar.gz',
    repos       =>  {
      'repos' => {
        'sentry' => {
          'url' => 'https://github.com/venmo/puppet-sentry.git',
        }, 
        'consulr' => {
          'url' => 'https://github.com/venmo/puppet-consulr.git',
        }, 
      },
    },
  }
```

## Proxy with Nginx
Using [jfryman/nginx](https://forge.puppetlabs.com/jfryman/nginx) puppet module you can easily proxy:

```
include nginx

nginx::resource::vhost { 'hound.something.tld':
  proxy => 'http://172.0.0.1:6080',
}
```

You can also extend this to terminate SSL connections.

## Adding repos
Every option under `repos` can be used, see the [example](https://github.com/etsy/Hound/blob/master/config-example.json) for possible options.

**PUPPET**
```
class { '::hound':
...
  repos =>
    "repos" => {
      "AnotherGitRepo" => {
          "url" => "https://www.github.com/YourOrganization/RepoOne.git",
          "ms-between-poll"   => 10000,
          "exclude-dot-files" => true,
      },
      "SomeMercurialRepo" ==> {
          "url" => "https://www.example.com/foo/hg",
          "vcs" => "hg",
      },
      "Subversion" => {
          "url" => "http://my-svn.com/repo",
          "url-pattern" => { 
              "base-url" => "{url}/{path}{anchor}",
          },
          "vcs" => "svn",
      },
    },
  },
...
}
```

**JSON** equivalent
```
"repos" : {
  "AnotherGitRepo" : {
      "url" : "https://www.github.com/YourOrganization/RepoOne.git",
      "ms-between-poll": 10000,
      "exclude-dot-files": true
  },
  "SomeMercurialRepo" : {
      "url" : "https://www.example.com/foo/hg",
      "vcs" : "hg"
  },
  "Subversion" : {
      "url" : "http://my-svn.com/repo",
      "url-pattern" : { 
          "base-url" : "{url}/{path}{anchor}"
      },
      "vcs" : "svn"
  }
}
```

## Fetch hound package
The hound team doesn't seem to provide a link to the binaries, so I built one on ubuntu precise64 machine, its located in the `tarballs` directory. These tarballs will **not** be available as part of the puppet module, you can only download them directly from the repo. In other words, you have to download the tarball to an accessible location and point `$hound::package_url` to that location.

## Managed vs Unmanaged
* *Managed config.json*
  * Is generated by puppet
  * `$hound::max_concurrent_indexers` and `$hound::repos` are only used when `$hound::managed_config` is `true`.
* *Unmanaged config.json*
  * Is generated by an external script, not by puppet.
  * `config.json` must be dropped in `${hound::conf_dir}`.
  * Although not generated by puppet, `config.json` must still be dropped in the same location as defined in puppet. Puppet run will fail if it can't find `config.json`.
  * Don't forget `dbpath` and `max-concurrent-indexers` when generating the config.json ([example](https://github.com/etsy/Hound/blob/master/config-example.json)).
  * `dbpath` must match `$hound::data_dir`.
  * `houndd` needs to be restarted when a new `config.json` is generated. Your external script can write `1` to `${hound::tmp_dir}/houndd_restart` and on the next puppet run `houndd` will be restarted.
  * You can also restart the service from the script `service houndd restart` if you choose not to use `houndd_restart` method.

## Limitations
* Only works with Ubuntu OS
* Only AMD64 arch is supported
* Only `tar.gz` packages are supported

## TODO
* unit tests

## Development
You know the deal: fork and pull
