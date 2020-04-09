# nethserver-business
Business Cube 2 integration for NethServer.

You can install this package from Software Center or with:

    yum install -y nethserver-business --enablerepo=nethforge


Database example:

    business=configuration
        ArcprocDB=Arcproc
        CompanyDB=Prova
        SharedFolder=


Events and actions
==================

`nethserver-business-upadte`: initialize default database.

`nethserver-business-conf-db`: import default Business Cube 2 databases.


Cockpit interface
=================

From Cockpit interface you can:
* connect and disconnect Business share: this share must be empty before starts installaton
* import and connect default databases to SQL Server
* check Business updates number
* view server logs

Once package is configured you can start Business Cube 2 installation using created Shared Folder and without installing databases because you've already connected them.
