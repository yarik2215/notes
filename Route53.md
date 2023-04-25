# Route53

## Notes

- The `A` record maps a name to one or more IP addresses when the IP are known and stable.
- The `CNAME` record maps a name to another name. It should only be used when there are no other records on that name.
- The `ALIAS` record maps a name to another name, but can coexist with other records on that name.
- The `URL` record redirects the name to the target name using the HTTP 301 status code.
