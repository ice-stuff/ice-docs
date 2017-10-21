# Instance registration

The following Bash lines are registering a running instance to iCE:

```bash
curl -L https://dl.bintray.com/glestaris/iCE/v2.1.0-rc.1/ice-agent \
    -O ./ice-agent
chmod +x ./ice-agent
./ice-agent register-self \
    --api-endpoint <iCE server URL> \
    --session-id <Session id>
```

The **iCE registry URL** is the URL of the iCE registry (e.g.:
`http://ice.example.org:8080`). The session id identifies a unique iCE session.
It can be obtained when starting the iCE shell, or through the iCE client
Python API.
