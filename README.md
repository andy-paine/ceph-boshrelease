# Ceph BOSH release

BOSH release for the distributed storage system [Ceph](https://ceph.io/). It uses `bosh export-release` to package Ceph for offline running.

## Usage
Ceph is mostly distributed via apt packages with a deep dependency tree. This makes it hard to write a packaging script that can "compile" the software in a way that allows it to be run offline without running `apt install`.

The solution used in this release is to download all the entire dependency graph of `ceph` as a set of debian packages and provide a script to install them in the right order. This allows the release to be compiled online where it can reach the correct `apt` repos.

## Building
To build this release for for offline environments, it needs to be compiled by deploying it to a BOSH director and then running `bosh export-release` for a specific stemcell. An example manifest can be found in [ci/files/compile.yml](ci/files/compile.yml).

```
bosh create-release
bosh upload-release
bosh deploy -d compiling-ceph ci/files/compile.yml
bosh export-release ceph/<version> ubuntu-xenial/<stemcell-version>
```

This will output a tarball - `ceph-<version>-ubuntu-xenial-<stemcell-version>-XXXXXXXX-XXXXXX-XXXXXX.tgz` which will contain all the relevant tarballs and a script to install them for offline use. The jobs then install Ceph in pre-start using:
```bash
pushd /var/vcap/packages/ceph
  dpkg -l ceph || ./install_ceph.sh
popd
```