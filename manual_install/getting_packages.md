# Obtaining Chef packages

Chef Inc. currently stores its RPM and DEB packages in [Package Cloud](https://packagecloud.io/chef)

We recommend you use [stable](https://packagecloud.io/chef/stable) rather than the more bleeding edge [current](https://packagecloud.io/chef/current) packages.

## Manual Download method

* From somewhere that can access the internet click the relevant package(s) and save local.
* From somewhere that can access your server(s), copy the package(s) to the server.

Use ```rpm -Uvh <rpm_filename>``` to install RPM packages on Redhat based distros.

Use ```dpkg -i <deb_filename>``` to install DEB packages on Debian based distros.

Windows users can use the tools in [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) or [FileZilla](https://filezilla-project.org/) to copy the packages to Linux servers using SCP or SFTP protocols.

## YUM Repository method
If your node has internet access you can tell YUM where to find the Chef package repository.

    curl -s https://packagecloud.io/install/repositories/chef/stable/script.rpm.sh | sudo bash

You can then install packages with

    yum install <package> -y

You may need to use an http/ftp proxy if you are behind a firewall.
If your proxy was ```my.internet.proxy``` listening on port ```3128``` then you could enter the following at the command prompt to cause YUM to use the proxy:-

    export HTTP_PROXY='http://my.internet.proxy:3128'
    export HTTPS_PROXY='http://my.internet.proxy:3128'
    export FTP_PROXY='http://my.internet.proxy:3128'

You can also configure the proxy permanently into your YUM configuration at ```/etc/yum.conf```

    proxy=http://my.internet.proxy:3128
    # Uncomment these lines if your proxy requires authentication
    # proxy_username=yumacc
    # proxy_password=clydenw

## APT Repository method
If your node has internet access you can tell APT where to find the Chef package repository.

    curl -s https://packagecloud.io/install/repositories/chef/stable/script.deb.sh | sudo bash

You can then install packages using

    apt-get install <package> -y

You may need to use an http/ftp proxy if you are behind a firewall.
If your proxy was ```my.internet.proxy``` listening on port ```3128``` then you could enter the following at the command prompt to cause APT to use the proxy:-

    export HTTP_PROXY='http://my.internet.proxy:3128'
    export HTTPS_PROXY='http://my.internet.proxy:3128'
    export FTP_PROXY='http://my.internet.proxy:3128'

You can also configure the proxy permanently into your APT configuration at ```/etc/apt/apt.conf```

    # Using a proxy with no authentication
    Acquire::http::Proxy "http://my.internet.proxy:3128";

    # Using a proxy that requires authentication
    Acquire::http::Proxy "http://username:password@my.internet.proxy:3128";
