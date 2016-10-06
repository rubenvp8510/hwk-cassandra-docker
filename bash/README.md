# Use

- hawkular -l: Launch seed node + hawkular-services on local
- hawkular -a user dest_host: Launch new cassandra node on <dest_host>
- hawkular -d user dest_host: Stop cassandra node on <dest_host>
- hawkular -r user dest_host:Delete cassandra node on <dest_host>

# TODO
  - When a node is removed this doesn't do the decomission process, so you can lost data.
  - The process ask the password twice (for ssh and sudo..) we need to improve this.
