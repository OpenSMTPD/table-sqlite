TABLE\_SQLITE(5) - File Formats Manual

# NAME

**table\_sqlite** - format description for smtpd sqlite tables

# DESCRIPTION

This manual page documents the file format of sqlite tables used by the
smtpd(8)
mail daemon.

The format described here applies to tables as defined in
smtpd.conf(5).

# SQLITE TABLE

A sqlite table allows the storing of usernames, passwords, aliases, and domains
in a format that is shareable across various machines that support
sqlite3(1)
(SQLite version 3).

The table is used by
smtpd(8)
when authenticating a user, when user information such as user-id and/or
home directory is required for a delivery, when a domain lookup may be required,
and/or when looking for an alias.

A sqlite table consists of one or more
sqlite3(1)
databases with one or more tables.

If the table is used for authentication, the password should be
encrypted using the
crypt(3)
function.
Such passwords can be generated using the
encrypt(1)
utility or
smtpctl(8)
encrypt command.

# SQLITE TABLE CONFIG FILE

The following configuration options are available:

**dbpath**
*file*

> This is the path to where the DB file is located.
> For example:

> > dbpath /etc/mail/smtp.sqlite

**query\_alias**
*SQL statement*

> This is used to provide a query to look up aliases.
> The question mark is replaced with the appropriate data.
> For alias it is the left hand side of the SMTP address.
> This expects one VARCHAR to be returned with the user name the alias
> resolves to.

**query\_credentials**
*SQL statement*

> This is used to provide a query for looking up user credentials.
> The question mark is replaced with the appropriate data.
> For credentials it is the left hand side of the SMTP address.
> The query expects that there are two VARCHARS returned, one with a user
> name and one with a password in
> crypt(3)
> format.

**query\_domain**
*SQL statement*

> This is used to provide a query for looking up a domain.
> The question mark is replaced with the appropriate data.
> For the domain it would be the right hand side of the SMTP address.
> This expects one VARCHAR to be returned with a matching domain name.

**query\_mailaddrmap**
*SQL statement*

> This is used to provide a query for looking up a senders.
> The question mark is replaced with the appropriate data.
> This expects one VARCHAR to be returned with the address the sender is
> allowed to send mails from.

A generic SQL statement would be something like:

	query_ SELECT value FROM table WHERE key=?;

# FILES

*/etc/mail/smtp.sqlite*

> Suggested
> sqlite3(1)
> database file.

*/etc/mail/sqlite.conf*

> Default
> table-sqlite(8)
> configuration file.

# EXAMPLES

Example based on the OpenSMTPD FAQ: Building a Mail Server
The filtering part is excluded in this example.
The configuration below is for a medium-size mail server which handles
multiple domains with multiple virtual users and is based on several
assumptions.
One is that a single system user named vmail is used for all virtual users.
This user needs to be created:

	# useradd -g =uid -c "Virtual Mail" -d /var/vmail -s /sbin/nologin vmail
	# mkdir /var/vmail
	# chown vmail:vmail /var/vmail

The sqlite schema is:

	CREATE TABLE virtuals (
	    id INTEGER PRIMARY KEY AUTOINCREMENT,
	    email VARCHAR(255) NOT NULL,
	    destination VARCHAR(255) NOT NULL
	);
	CREATE TABLE credentials (
	    id INTEGER PRIMARY KEY AUTOINCREMENT,
	    email VARCHAR(255) NOT NULL,
	    password VARCHAR(255) NOT NULL
	);
	CREATE TABLE domains (
	    id INTEGER PRIMARY KEY AUTOINCREMENT,
	    domain VARCHAR(255) NOT NULL
	);

Which can be populated as follows:

	INSERT INTO domains VALUES (1, "example.com");
	INSERT INTO domains VALUES (2, "example.net");
	INSERT INTO domains VALUES (3, "example.org");
	
	INSERT INTO virtuals VALUES (1, "abuse@example.com", "bob@example.com");
	INSERT INTO virtuals VALUES (2, "postmaster@example.com", "bob@example.com");
	INSERT INTO virtuals VALUES (3, "webmaster@example.com", "bob@example.com");
	INSERT INTO virtuals VALUES (4, "bob@example.com", "vmail");
	INSERT INTO virtuals VALUES (5, "abuse@example.net", "alice@example.net");
	INSERT INTO virtuals VALUES (6, "postmaster@example.net", "alice@example.net");
	INSERT INTO virtuals VALUES (7, "webmaster@example.net", "alice@example.net");
	INSERT INTO virtuals VALUES (8, "alice@example.net", "vmail");
	
	INSERT INTO credentials VALUES (1, "bob@example.com", "$2b$08$ANGFKBL.BnDLL0bUl7I6aumTCLRJSQluSQLuueWRG.xceworWrUIu");
	INSERT INTO credentials VALUES (2, "alice@example.net", "$2b$08$AkHdB37kaj2NEoTcISHSYOCEBA5vyW1RcD8H1HG.XX0P/G1KIYwii");

The
*/etc/mail/sqlite.conf*
file then specifies the following queries:

	dbpath /etc/mail/smtp.sqlite
	query_alias SELECT destination FROM virtuals WHERE email=?;
	query_credentials SELECT email, password FROM credentials WHERE email=?;
	query_domain SELECT domain FROM domains WHERE domain=?;

And the following configuration for
smtpd(8)
in
*/etc/mail/smtpd.conf*:

	table domains sqlite:/etc/mail/sqlite.conf
	table virtuals sqlite:/etc/mail/sqlite.conf
	table credentials sqlite:/etc/mail/sqlite.conf
	listen on egress port 25 tls pki mail.example.com
	listen on egress port 587 tls-require pki mail.example.com auth <credentials>
	accept from any for domain <domains> virtual <virtuals> deliver to mbox

# TODO

Documenting the following query options:

	**query_netaddr**
	**query_userinfo**
	**query_source**
	**query_mailaddr**
	**query_addrname**

# SEE ALSO

encrypt(1),
crypt(3),
smtpd.conf(5),
smtpctl(8),
smtpd(8)

Nixpkgs - April 20, 2024
