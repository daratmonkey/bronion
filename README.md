# bronion
## Import Bro logs from SecurityOnion into Logstash

The files in this repo are intended to create a modular workflow that is easy to modify and debug.
The normal logstash data pipeline consists of *input->filter->output*. Normally all filters, inputs, and outputs
are connected unless you control the message routing with tags.

Clone the repo into your logstash config directory and build your configuration from softlinks. To create a workflow
you need to have an input, output, filter, and output. It uses the same idea Debian has for Apache configuration

Since Bro has many different file types, a partial match is done to classify the Bro log type (20_filter-preprocessor-bro.conf). This partial match is used then to process the
log with an appropiate filter

The postprocessor filter deletes any entry with the *_grokparsefailure* tag. 
If you are debugging or developing new filters delete this postprocessor temporarly

---

## Example

You want to ingest *bro_conn* logs from the SecurityOnion into Logstash. The first step is to modify *syslog-ng.conf*
in the sensor that has the logs you want to ingest. The destination IP address is the host where logstash is running.

```
destination d_bronion { udp("1.2.3.4" port("5514"));};
log {
  source(s_bro_conn);
  rewrite(r_pipes);
  destination(d_bronion);
};
```

Then proceed to build your configuration on the host that is running logstash

```shell
cd /etc/logstash/conf.d/
ln -s policy/10_input-udp-5514.conf conf.d/10_input-udp-5514.conf
ln -s policy/20_filter-preprocessor-bro.conf conf.d/20_filter-preprocessor-bro.conf
ln -s policy/21_filter-bro-conn.conf conf.d/policy/21_filter-bro-conn.conf
ln -s policy/29_filter-postprocessor-bro.conf conf.d/29_filter-postprocessor-bro.conf
ln -s policy/30_output-elasticsearch.conf conf.d/30_output-elasticsearch.conf
service logstash restart
```
