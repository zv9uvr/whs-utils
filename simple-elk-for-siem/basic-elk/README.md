# Basic ELK

Minimal three-node Elasticsearch and Kibana lab stack for SIEM practice.

## Version

- Elastic Stack: 9.4.2
- Elasticsearch: 9.4.2
- Kibana: 9.4.2

## Run

```bash
docker compose up
```

Kibana: <http://localhost:5601>

Default login:

- Username: `elastic`
- Password: `p@ssw0rd1234`

## Notes

- This stack is for local lab use, not production.
- Elasticsearch runs as three nodes with reduced lab resource limits.
- The compose file disables mmap-backed storage so students do not need to tune `vm.max_map_count` before starting the lab.
