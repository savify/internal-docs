# How to setup Kibana logs view

If you want to review system logs in more pretty way, you can use Kibana.

## How to show logs in Kibana

1. Go to **[Kibana - http://localhost:5601/](http://localhost:5601/)** and open **Discover** page:
![Kibana Discover Tab](kibana-discover-tab.png)
2. Create a new Data View, enter view name and index pattern that will match `savify-[ENV]` index:
![Kibana Create Data View](kibana-create-data-view.png)
3. Now you have a Logs dashboard - you can add fields, search by them and many many more:
![Kibana Logs](kibana-logs.png)
