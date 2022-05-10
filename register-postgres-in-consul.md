# Register Postgres service in Consul with Ansible

Registering a service in Consul is easy. Consul [provides](https://www.consul.io/commands/services/register) a command
to do it from CLI (`consul services register`). Ansible provides `community.general.consul` to interact with Consul,
but it requires all hosts to have `python-consul` and `python-requests` packages installed. To work around this 
requirement, use Consul's [ReST API to register the service](https://www.consul.io/api-docs/agent/service#register-service).

## Healthcheck
Postgres does not have a built in health check. Instead, we will check if the TCP port is open. Consul has a built in 
[TCP check](https://www.consul.io/api-docs/agent/check#tcp).

## API
Endpoint: `PUT` `/agent/service/register`

Body of the request:
```json
{
  "id": "postgres-nog",
  "name": "postgres",
  "port": 5432,
  "check": {
    "name": "Postgres port check",
    "tcp": "localhost:5432",
    "interval": "10s",
    "timeout": "2s"
  }
}
```

Ansible's `ansible.builtin.uri` module provides the functionality to make the API call.
```yaml
- name: Register service in Consul
  ansible.builtin.uri:
    url: "http://localhost:8500/v1/agent/service/register"
    method: PUT
    body: "{{ lookup('template', 'templates/register-consul.json.j2', convert_data=True) | to_json }}"
```

tags: #homelab, #postgres, #consul
