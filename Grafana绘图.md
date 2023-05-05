技能

+ 

我是一名资深的DevOps工程师，拥有丰富的Prometheus和Grafana实战经验。我熟练掌握Prometheus的基本概念、架构、指标类型及其监控方法。熟知PromQL语法，能够编写高效而精准的查询语句，并通过Alertmanager实现告警。

同时，我也具备了解和配置Grafana的能力，可以根据业务需求设计并优化可视化仪表板，并能够结合Prometheus数据源完成相关查询和展示。在实践中，我常常使用Prometheus和Grafana来监测系统性能、网络质量、服务稳定性等方面的指标，以便更好地保障产品和服务的正常运行，提高用户体验。



```JSON
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "target": {
          "limit": 100,
          "matchAny": false,
          "tags": [],
          "type": "dashboard"
        },
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": 30,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "datasource": {
        "type": "datasource",
        "uid": "grafana"
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 0,
        "y": 0
      },
      "id": 8,
      "options": {
        "bgColor": "",
        "clockType": "custom",
        "countdownSettings": {
          "endCountdownTime": "2023-04-13T02:20:54+08:00",
          "endText": "00:00:00"
        },
        "countupSettings": {
          "beginCountupTime": "2023-04-13T02:20:54+08:00",
          "beginText": "00:00:00"
        },
        "dateSettings": {
          "dateFormat": "YYYY-MM-DD",
          "fontSize": "20px",
          "fontWeight": "normal",
          "locale": "",
          "showDate": true
        },
        "mode": "time",
        "refresh": "sec",
        "timeSettings": {
          "fontSize": "20px",
          "fontWeight": "normal"
        },
        "timezone": "",
        "timezoneSettings": {
          "fontSize": "12px",
          "fontWeight": "normal",
          "showTimezone": false,
          "zoneFormat": "offsetAbbv"
        }
      },
      "pluginVersion": "2.1.3",
      "title": "time",
      "type": "grafana-clock-panel"
    },
    {
      "datasource": {
        "type": "datasource",
        "uid": "grafana"
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 3,
        "y": 0
      },
      "id": 26,
      "options": {
        "code": {
          "language": "plaintext",
          "showLineNumbers": false,
          "showMiniMap": false
        },
        "content": "\n<p>这是一个项目名</p>",
        "mode": "html"
      },
      "pluginVersion": "9.3.2",
      "title": "项目名",
      "type": "text"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "max": 100,
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue"
              },
              {
                "color": "#EAB839",
                "value": 50
              },
              {
                "color": "dark-orange",
                "value": 70
              },
              {
                "color": "dark-red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 6,
        "y": 0
      },
      "id": 18,
      "options": {
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "count(kube_pod_status_phase{phase=\"Running\"})",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Panel Title",
      "type": "gauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 40
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 9,
        "y": 0
      },
      "id": 2,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "text": {
          "valueSize": 50
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "count(kube_pod_status_phase{phase=\"Running\"})",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "集群运行中Pod数量",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 2,
        "x": 12,
        "y": 0
      },
      "id": 4,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "text": {
          "valueSize": 50
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "sum(sum(node_memory_MemTotal_bytes) by (instance))",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "集群总内存",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "custom": {
            "align": "center",
            "displayMode": "auto",
            "filterable": true,
            "inspect": false
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              }
            ]
          }
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "Time"
            },
            "properties": [
              {
                "id": "custom.width",
                "value": 216
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "instance"
            },
            "properties": [
              {
                "id": "custom.width",
                "value": 196
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "node"
            },
            "properties": [
              {
                "id": "custom.width",
                "value": 117
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "kubelet_version"
            },
            "properties": [
              {
                "id": "custom.width",
                "value": 148
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "os_image"
            },
            "properties": [
              {
                "id": "custom.width",
                "value": 173
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 8,
        "w": 10,
        "x": 14,
        "y": 0
      },
      "id": 14,
      "options": {
        "footer": {
          "enablePagination": false,
          "fields": [
            "Value"
          ],
          "reducer": [
            "last"
          ],
          "show": false
        },
        "frameIndex": 1,
        "showHeader": true,
        "sortBy": [
          {
            "desc": true,
            "displayName": "node"
          }
        ]
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "sum(kube_node_info)by(kubelet_version,node,internal_ip,os_image,kernel_version,node)",
          "format": "table",
          "hide": false,
          "instant": true,
          "legendFormat": "",
          "range": false,
          "refId": "B"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "",
          "hide": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "节点信息",
      "type": "table"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [
            {
              "options": {
                "from": 1,
                "result": {
                  "index": 0,
                  "text": "ok"
                },
                "to": 50
              },
              "type": "range"
            },
            {
              "options": {
                "from": 100,
                "result": {
                  "index": 1,
                  "text": "error"
                },
                "to": 999
              },
              "type": "range"
            }
          ],
          "max": 100,
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue"
              },
              {
                "color": "#EAB839",
                "value": 50
              },
              {
                "color": "dark-orange",
                "value": 70
              },
              {
                "color": "dark-red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 0,
        "y": 5
      },
      "id": 19,
      "options": {
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "count(kube_pod_status_phase{phase=\"Running\"})",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Panel Title",
      "type": "gauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 3,
        "y": 5
      },
      "id": 10,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "text": {
          "valueSize": 20
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "sum(node_memory_MemTotal_bytes)",
          "hide": false,
          "legendFormat": "总内存",
          "range": true,
          "refId": "C"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "sum(sum(node_memory_MemAvailable_bytes)by(instance))",
          "legendFormat": "可用内存",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "内存",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-BlPu"
          },
          "mappings": [],
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue"
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 8,
        "x": 6,
        "y": 5
      },
      "id": 20,
      "options": {
        "displayMode": "gradient",
        "minVizHeight": 10,
        "minVizWidth": 0,
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "count by(namespace) (kube_pod_status_phase{phase=\"Running\"})",
          "format": "time_series",
          "instant": true,
          "legendFormat": "__auto",
          "range": false,
          "refId": "A"
        }
      ],
      "title": "命名空间下Pod数量",
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "custom": {
            "align": "center",
            "displayMode": "auto",
            "filterable": false,
            "inspect": true
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "Value"
            },
            "properties": [
              {
                "id": "custom.displayMode",
                "value": "lcd-gauge"
              },
              {
                "id": "color",
                "value": {
                  "mode": "continuous-BlPu"
                }
              },
              {
                "id": "displayName",
                "value": "已使用"
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Time"
            },
            "properties": [
              {
                "id": "displayName",
                "value": "id"
              },
              {
                "id": "unit",
                "value": "d"
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 7,
        "w": 10,
        "x": 14,
        "y": 8
      },
      "id": 16,
      "options": {
        "footer": {
          "enablePagination": false,
          "fields": "",
          "reducer": [
            "sum"
          ],
          "show": false
        },
        "frameIndex": 0,
        "showHeader": true,
        "sortBy": [
          {
            "desc": false,
            "displayName": "mountpoint"
          }
        ]
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "max by (mountpoint) (node_filesystem_avail_bytes {job=\"node-exporter\", instance=\"k8s-master01\", fstype!=\"\"})",
          "format": "table",
          "instant": true,
          "interval": "",
          "legendFormat": "",
          "range": false,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "hide": false,
          "refId": "B"
        }
      ],
      "title": "磁盘可用",
      "type": "table"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-BlPu"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "semi-dark-red",
                "value": 50
              }
            ]
          },
          "unit": "%"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 6,
        "x": 0,
        "y": 10
      },
      "id": 12,
      "options": {
        "colorMode": "background",
        "graphMode": "none",
        "justifyMode": "center",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "text": {
          "titleSize": 20,
          "valueSize": 25
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "sum(node_memory_MemFree_bytes / node_memory_MemTotal_bytes * 100) by(instance)",
          "legendFormat": "{{label_name}}",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "节点内存使用率",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            }
          },
          "mappings": [],
          "unit": "none"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 8,
        "x": 6,
        "y": 10
      },
      "id": 25,
      "options": {
        "displayLabels": [
          "name"
        ],
        "legend": {
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "pieType": "pie",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "count by(namespace) (kube_pod_status_phase{phase=\"Running\"})",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "节点Pod数",
      "type": "piechart"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineStyle": {
              "fill": "solid"
            },
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": true,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "decimals": 1,
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "%"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "memory"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "dark-red",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "cpu"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "dark-blue",
                  "mode": "fixed"
                }
              },
              {
                "id": "custom.axisPlacement",
                "value": "right"
              },
              {
                "id": "custom.fillOpacity",
                "value": 56
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 0,
        "y": 15
      },
      "id": 6,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "sum(avg(100*(1-(node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)))) by(instance) - 40",
          "legendFormat": "memory",
          "range": true,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "avg(100 - avg(irate(node_cpu_seconds_total{mode=\"idle\"}[5m]))by (instance) * 100)",
          "hide": false,
          "interval": "",
          "legendFormat": "cpu",
          "range": true,
          "refId": "B"
        }
      ],
      "title": "内存/CPU",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "datasource",
        "uid": "grafana"
      },
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 12,
        "y": 15
      },
      "id": 32,
      "options": {
        "alertInstanceLabelFilter": "",
        "alertName": "",
        "dashboardAlerts": false,
        "datasource": "-- Grafana --",
        "groupBy": [],
        "groupMode": "default",
        "maxItems": 20,
        "sortOrder": 1,
        "stateFilter": {
          "error": true,
          "firing": true,
          "noData": true,
          "normal": true,
          "pending": true
        },
        "viewMode": "list"
      },
      "title": "Panel Title",
      "type": "alertlist"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-BlPu"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byFrameRefID",
              "options": "B"
            },
            "properties": [
              {
                "id": "unit",
                "value": "%"
              },
              {
                "id": "min",
                "value": 0
              },
              {
                "id": "max",
                "value": 100
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 8,
        "w": 6,
        "x": 0,
        "y": 24
      },
      "id": 22,
      "options": {
        "displayMode": "lcd",
        "minVizHeight": 10,
        "minVizWidth": 0,
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "sum by(instance) (node_filesystem_avail_bytes{mountpoint=\"/\"} * 100 / node_filesystem_size_bytes{mountpoint=\"/\"}) ",
          "hide": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "B"
        }
      ],
      "title": "根分区可用率",
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-BlPu"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 6,
        "x": 6,
        "y": 24
      },
      "id": 23,
      "options": {
        "displayMode": "lcd",
        "minVizHeight": 10,
        "minVizWidth": 0,
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true
      },
      "pluginVersion": "9.3.2",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "max by (instance) (node_filesystem_avail_bytes {job=\"node-exporter\",mountpoint=\"/\"})",
          "hide": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "B"
        }
      ],
      "title": "根分区可用空间",
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "P1809F7CD0C75ACF3"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-BlPu"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisGridShow": false,
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 11,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "smooth",
            "lineStyle": {
              "fill": "solid"
            },
            "lineWidth": 2,
            "pointSize": 3,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 24
      },
      "id": 30,
      "options": {
        "legend": {
          "calcs": [
            "last",
            "min",
            "max"
          ],
          "displayMode": "table",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "desc"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "P1809F7CD0C75ACF3"
          },
          "editorMode": "code",
          "expr": "sum by(instance)(node_filesystem_avail_bytes{mountpoint=\"/\",instance=\"$nodename\"})",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Panel Title",
      "type": "timeseries"
    }
  ],
  "refresh": "30s",
  "schemaVersion": 37,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": [
      {
        "current": {
          "selected": false,
          "text": "k8s-master02",
          "value": "k8s-master02"
        },
        "datasource": {
          "type": "prometheus",
          "uid": "P1809F7CD0C75ACF3"
        },
        "definition": "label_values(kube_pod_info, node)",
        "hide": 0,
        "includeAll": false,
        "label": "nodename",
        "multi": false,
        "name": "nodename",
        "options": [],
        "query": {
          "query": "label_values(kube_pod_info, node)",
          "refId": "StandardVariableQuery"
        },
        "refresh": 1,
        "regex": "/.*(k8s.*)/",
        "skipUrlSync": false,
        "sort": 0,
        "type": "query"
      }
    ]
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "Project",
  "uid": "rmZDKjLVz",
  "version": 30,
  "weekStart": ""
}
```

