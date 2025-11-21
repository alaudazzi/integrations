# Service Info

## Common use cases

ModSecurity is a Web Application Firewall (WAF) that can be used to protect web applications from a wide range of attacks, including the OWASP Top Ten. Common use cases include:
- Real-time monitoring, logging, and analysis of HTTP/S traffic.
- Blocking common web application attacks such as SQL injection, cross-site scripting (XSS), and local file inclusion (LFI).
- Virtual patching of known application vulnerabilities.
- Compliance with regulations like PCI DSS.

## Data types collected

This integration collects ModSecurity audit logs. These logs contain detailed information about each web request that is inspected by ModSecurity, including:
- Request and response headers
- Request body
- Details of any matched security rules
- Transaction metadata

The data is collected into the `auditlog` data stream.

## Compatibility

This integration has been tested with:
- ModSecurity v3 with the nginx connector.
- ModSecurity v3 with the Apache connector.

The integration requires the ModSecurity audit log format to be set to `JSON`.

## Scaling and Performance

The performance of ModSecurity largely depends on the number and complexity of the rules that are enabled. For high-traffic environments, it is important to:
- Only enable rules that are relevant to your application stack.
- Use a performance-optimized ruleset, such as the OWASP Core Rule Set (CRS).
- Monitor the performance of your web server and adjust the ModSecurity configuration as needed.

# Set Up Instructions

## Vendor prerequisites

- A working ModSecurity v3 installation with either the Apache or nginx connector.
- The OWASP Core Rule Set (CRS) is recommended for a baseline of security rules.

## Elastic prerequisites

There are no specific Elastic prerequisites beyond a working Elastic Agent to collect the logs.

## Vendor set up steps

To send data to Elastic, you must configure ModSecurity to write audit logs in JSON format. Add the following directives to your ModSecurity configuration:

```
SecAuditLogParts ABDEFHIJZ
SecAuditLogType Serial
SecAuditLog /var/log/modsec_audit.json
SecAuditLogFormat JSON
```

**Note:** It is important to *exclude* the `K` part from `SecAuditLogParts`. Including it can make the raw logs too long for the ingest pipeline to parse. The log path can be adjusted as needed, but ensure the Elastic Agent has read permissions to it.

## Kibana set up steps

1.  In Kibana, navigate to **Management > Integrations**.
2.  Search for "ModSecurity" and click on it.
3.  Click **Add ModSecurity Audit**.
4.  Configure the integration name and optionally an agent policy.
5.  Under **Settings**, provide the path to the ModSecurity audit log file (e.g., `/var/log/modsec_audit.json`).
6.  Click **Save and continue**. This will deploy the necessary assets (like dashboards and ingest pipelines) and update the specified agent policy to start collecting the logs.

# Validation Steps

1.  After configuring the integration, trigger some ModSecurity rules. You can do this by sending a malicious-looking request to your web server, for example: `curl 'http://localhost/?param="><script>alert(1)</script>'`.
2.  In Kibana, navigate to the **Discover** app.
3.  Check the `logs-modsecurity.auditlog-*` data stream for incoming documents. You should see events corresponding to the requests you sent.
4.  Navigate to **Dashboard** and search for the "ModSecurity" dashboard to see visualizations of your data.

# Troubleshooting

## Common Configuration Issues

*   **Issue**: No data is being collected.
    *   **Solution**:
        *   Verify that the Elastic Agent is running and has the correct permissions to read the ModSecurity audit log file.
        *   Check the path to the audit log file in the integration settings in Kibana.
        *   Ensure that ModSecurity is generating audit logs. Check the contents of the log file directly.
        *   Verify that the web server is receiving traffic that would trigger ModSecurity rules.

## Ingestion Errors

*   **Issue**: Logs are present in the log file but not appearing in Elasticsearch, and there are errors in the agent logs.
    *   **Solution**: This is often due to the log line being too long. Ensure that you have excluded the `K` part from the `SecAuditLogParts` directive in your ModSecurity configuration. If you need parts of the `K` field, you may need to customize the ingest pipeline to handle larger log sizes.

## Vendor Resources

- [ModSecurity GitHub Repository](https://github.com/SpiderLabs/ModSecurity)
- [OWASP ModSecurity Core Rule Set](https://coreruleset.org/)

# Documentation sites

- [ModSecurity Reference Manual](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual-(v2.x)) (Note: This is for v2.x but much of it is still relevant for v3)
- [OWASP ModSecurity Core Rule Set Documentation](https://coreruleset.org/docs/)
