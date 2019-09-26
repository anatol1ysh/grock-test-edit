# NGinX - Version (Stable) -> 1.16.1
# Apache httpd - Version (Stable) -> 2.4.41

# outside the docker-compose run the nginx in dedicated mode use Docker CLI:

docker run -d -p 8080:80 --log-driver gelf --log-opt gelf-address=udp://localhost:12201 --name=nginx nginx:1.16.1

# steps to actions:
  - configure come message by python script
  - test cutom scripts for grok in elastic stack

# ___ some links about:
https://logz.io/blog/logstash-grok/
https://logz.io/blog/logstash-mutate-filter/
https://www.elastic.co/guide/en/logstash/2.3/plugins-filters-grok.html

# ___ logstash:
https://www.elastic.co/guide/en/logstash/current/field-extraction.html

# ___ elastic _ regexp:
https://www.elastic.co/guide/en/elasticsearch/reference/7.3/regexp-syntax.html

# ___ 
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html

#analize prod input | filter | output ->
input {
    tcp {
        port => 4560
        codec => json_lines
    }
}
filter {
    grok {
        match => {
            "message" => "\"message\":%{GREEDYDATA:message},\"logdata\":%{GREEDYDATA:logdata},\"fields\":%{GREEDYDATA:fields}}"
        }
        overwrite => ["message"]
    }
    json {
        source => "fields"
        target => "parsed_fields"
        remove_field => ["fields"]
    }
}
output {
    elasticsearch {
        hosts => ["elasticsearch"]
        manage_template => false
        index => "logback-%{+YYYY.MM.dd}"
        document_type => "logback-json-log"
    }
    stdout {
        codec => rubydebug
    }
}