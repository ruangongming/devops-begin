# Activate Atlassiasn Software



Open Application on browser

Enter the requested information, When asked for license

run `atlassian-agent.jar` in Application container


docker exec **`<container_name>`** java -jar atlassian-agent.jar -m **`<email>`** -o **`<organisation>`** -p **`<product>`** -s **`<ServerID>`**

Server ID is shown at License request step on **Application Setup wizard** 

Copy ***Produced license*** to ***License request field*** & Click OK

Enjoy :)


## ACTIVE DATACENTER MODE

**Support for Atlassiasn Server products ends on Feb. 15, 2024, Use "`-d`" parameter for Generate Datacenter license**


## example

```bash
docker exec confluence java -jar /atlassian-agent.jar -d -m a@gmail.com -o a -p conf -s BERU-UNDU-YT8Q-KRND
```


## -d,--datacenter
Data center license

## -m,--mail
License email

## -o,--organisation
License organisation


## -p,--product
License product

List of Product Support

- `jc` : **JIRA Core**
- `jsd` : **JIRA Service Desk**
- `jira`: **JIRA Software**
- `jsm`: **JIRA Service Management**
- `crucible` : **Crucible**
- `conf` : **Confluence**
- `bitbucket` : **Bitbucket**
- `bamboo` : **Bamboo**
- `fisheye` : **FishEye**
- `questions`: **Questions plugin for Confluence**
- `crowd` : **Crowd**
- `capture`: **Capture plugin for JIRA**
- `training`: **Training plugin for JIRA**
- `tc`: **Team Calendars plugin for Confluence**
- `portfolio` : **Portfolio plugin for JIRA**
- `*` : **Third party plugin key**


