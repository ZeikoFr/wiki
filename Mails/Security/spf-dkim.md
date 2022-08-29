# SPF

## What is SPF
SPF Record is a DNS TXT record

 - MUST start with "v=spf1"
 - MUST end with "all" keyword

## Practical example

For a simple example we will demonstrate SPF records with multiple sender for the domain **random.org** :

 - Mail server : 60.100.200.10
 - Web servers : 60.100.200.20 and 60.100.200.21
 - Remote 3rd party "mailer.com" : 180.210.23.4 2607:DA00:0401::7777

We also have the following records in place on **random.org** :

 - random.org A 60.100.200.20
 - random.org MX mail.random.org
 - mail.random.org A 60.100.200.10
 - wiki.random.org A 60.100.200.21

Here is our SPF record : `v=spf1 ip4:60.100.200.10 ip4:60.100.200.20 ip4:60.100.200.21 ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`

the beginning and end have been explained at the top of this page.

The `ip4:` indicate an IPv4 address (seems logic), same for the `ip6:`

The symbol before `all` have a specific meaning :

 - `?` means "neutral" ie : No policy is enabled. **Do not use this** 

 - `~` means "soft fail" ie : Any server not listed in the SPF that send mail as **"@random.org"** should be treated as possible spam.

 - `-` means "hard fail" ie : Any mail that is NOT comming from the listed IP should be treated has Junk/Spam.

So far our record contain 4 IPv4 and 1 IPv6.

### Factoring SPF records

But since we send and receive email from the same server we can simplify our record by using the "mx" value.

 - Previous record : `v=spf1 ip4:60.100.200.10 ip4:60.100.200.20 ip4:60.100.200.21 ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`
 - New record : `v=spf1 mx ip4:60.100.200.20 ip4:60.100.200.21 ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`

The recepient server will now automaticly lookup the MX record and determine that it is allowed to send email.

In the future if we add other MX records we do not have to modify SPF record has `mx` value cover all MX records.

> In case the receiving server DOES NOT send email (an antispam appliance or else), then do not add MX in the SPF record.
{.is-warning}



Similar to the `mx` value, the `a` value allow us to remove the IP of our web server to use the DNS A record.

>The `a` value only apply to the random.org A and AAAA records, not to other record like wiki.record.org
{.is-info}

 - Previous record : `v=spf1 mx ip4:60.100.200.20 ip4:60.100.200.21 ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`
 - New record : `v=spf1 mx a ip4:60.100.200.21 ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`



But we also send email from wiki.random.org, and we don't want to modify SPF if we change the IP of the A record.

 - Previous record : `v=spf1 mx a ip4:60.100.200.21 ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`
 - New record : `v=spf1 mx a a:wiki.random.org ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`

So now we are left with the 3rd party vendor IP addresses authorized to send email in **random.com** name.

But in case of IP change vendor side we need to modify the SPF record again !

We can recursively allow the SPF of mailer.com in our SPF record :

 - Previous record : `v=spf1 mx a a:wiki.random.org ip4:180.210.23.4 ipv6:2607:DA00:0401::7777 ~all`
 - New record : `v=spf1 mx a a:wiki.random.org include:_spf.mailer.com ~all`

> Always check with your 3rd party what is the proper include to do.
{.is-warning}



And finally we can enforce our policy by switching from `~all` to `-all`.

This is the "hard fail" which mean : Any mail that is NOT comming from the listed IP should be treated has Junk/Spam.

It is still up to the recepient to either follow or ignore the SPF record.

To address this security problem a standard named DMARC is used.

# SPF best practice

 - Only 1 record per domain.
 - No private ip in record.
 - You can leverage DMARC rua report to get list of ip sending mail in your behalf.
 - You can use `~` temporarly to get time to compile properly the full ip list.
 - When using **include:** your `all` value take the priority, so if you have `-all` and the include use `?all` the enforced value in `-all`.
 - Protect unused domain with empty SPF ! : `v=spf1 -all`.
 - Avoid listing large block of IP in SPF
 - Be careful about lookup limitation, only 10 lookups are allowed (a, mx, and include: are lookups)
 - Be careful of length limits of SPF, no more than 255 characters
  
# DKIM

## What is DKIM

## Create a DKIM key

 - Create a pair of Pubkey/Privkey (Either with the solution provided by your mail server or by a simple open SSL command)
 - Put the pubkey in a DNS record

 Example : 
   - For an RSA key `openssl genrsa -out private-rsa.key 2048 && openssl rsa -in private-rsa.key -pubout -out public-rsa.key`
   - For an ed25519 key `openssl genpkey -algorithm ed25519 -out private-ed25519.pem && openssl pkey -in private-ed25519.pem -pubout -out private-ed25519.pem`

Be carefull has ed25519 has not spread out for now, if possible use double signing (rsa+ed25519) to avoid trouble of server not checking ed25519.

## Choose a selector

The selector can be anything of your choice as long as it is permitted in DNS record.

If it is automatically managed by your systems (from creation of the key to DNS setup), then the selector might be random value.

If you manage the DNS record manually then you can implement a standard for your own domain, example <service-month-year> like : `mailchimp-08-2022._domainkey.random.org`

Always ask the third-party if they're expecting a specific selector.

The critical thing is to have unique value for each selector, so when you rotate the mailchimp key, do not edit the current record. Instead create a new one following you standard, with the new key.

Then after a week (DNS is slow !!) Remove the old record.

## DKIM Best Practices

 - Use DKIM every server that send email from your domain name.
 - For servers that do not support DKIM, relay outbound email to another server that support it and let this server send the mail.
 - Verify your SPF after setting DKIM to be sure everything use DKIM.
 - Rotate the DKIM key on a scheduled basis
 - Make sure the DKIM private key is secure.
 - Unique DKIM key on each server makes troubleshooting easier.
