#Cloudify actionable events and New Relic

The guide describes how to configure Cloudify Manager to send events to New Relic insights.

#####The guide based on documentation:
* [Cloudify actionable events](https://docs.cloudify.co/5.0.5/working_with/manager/actionable-events/)
* [New Relic Event API](https://docs.newrelic.com/docs/insights/insights-data-sources/custom-data/introduction-event-api)

#####Pre-requireents:
* Plugin [cloudify-utilities-plugin]() should be uploaded to the cloudify manager

###Configuration

__File `/opt/mgmtworker/config/hooks.conf`__

_Hooks parameters:_
* event_type
* implementation
* inputs
  * logger_file
  * prerender
  * properties
    * hosts
    * port
    * ssl
    * verify
  * template_file
  * params
    * account_id
    * api_key
* description

See example in Appendix A

__File with REST call (template_file parameter in hooks.conf)__

It uses Jinja2 template and its parameters to render the REST CALL 
Also hooks context provides `__inputs__` dictionary. See the dictionary example bellow:
```json
'inputs': {
  'event_type': 'workflow_succeeded',
  'is_system_workflow': False,
  'blueprint_id': 'test',
  'tenant_name': 'default_tenant',
  'rest_token': 'WyIwIiwiJHBia2RmMi1zaGEyNTYkMjkwMDAkdkJkQ0tHWE0uVjlyN1IxakRPRmNpdyRzbENtU0E5SzRBS2FaR1czUXhFYW1kM1FZMFdmWHhsa054RWRtRW1NQjdzIl0.XpA8MA.cDcGa1WL_jZDhBtpGojJfWRm63s',
  'workflow_id': 'create_deployment_environment',
  'arguments': None,
  'timestamp': '2020-04-10T09:28:18.198Z',
  'deployment_id': 'test',
  'message_type': 'hook',
  'execution_id': 'dda92787-1f51-4549-9565-a12f3d987cba',
  'execution_parameters': {
    'deployment_plugins_to_install': [
      
    ],
    'workflow_plugins_to_install': [
      {
        'distribution_release': None,
        'install_arguments': None,
        'name': 'default_workflows',
        'package_name': None,
        'distribution_version': None,
        'package_version': None,
        'supported_platform': None,
        'source': None,
        'install': False,
        'executor': 'central_deployment_agent',
        'distribution': None
      }
    ],
    'policy_configuration': {
      'policy_types': {
        'cloudify.policies.types.host_failure': {
          'source': 'file:///opt/manager/resources/cloudify/policies/host_failure.clj',
          'properties': {
            'policy_operates_on_group': {
              'default': False,
              'description': 'If the policy should maintain its state for the whole group\nor each node instance individually.\n'
            },
            'is_node_started_before_workflow': {
              'default': True,
              'description': 'Before triggering workflow, check if the node state is started'
            },
            'service': {
              'default': [
                'service'
              ],
              'description': 'Service names whose events should be taken into consideration'
            },
            'interval_between_workflows': {
              'default': 300,
              'description': 'Trigger workflow only if the last workflow was triggered earlier than interval-between-workflows seconds ago.\nif < 0  workflows can run concurrently.\n'
            }
          }
        },
        'cloudify.policies.types.ewma_stabilized': {
          'source': 'file:///opt/manager/resources/cloudify/policies/ewma_stabilized.clj',
          'properties': {
            'is_node_started_before_workflow': {
              'default': True,
              'description': 'Before triggering workflow, check if the node state is started'
            },
            'ewma_timeless_r': {
              'default': 0.5,
              'description': 'r is the ratio between successive events. The smaller it is, the smaller impact on the computed value the most recent event has.\n'
            },
            'upper_bound': {
              'default': True,
              'description': "boolean value for describing the semantics of the threshold.\nif 'true': metrics whose value is bigger than the threshold will cause the triggers to be processed.\nif 'false': metrics with values lower than the threshold will do so.\n"
            },
            'stability_time': {
              'default': 0,
              'description': 'How long a threshold must be breached before the triggers will be processed'
            },
            'policy_operates_on_group': {
              'default': False,
              'description': 'If the policy should maintain its state for the whole group\nor each node instance individually.\n'
            },
            'service': {
              'default': 'service',
              'description': 'The service name'
            },
            'threshold': {
              'description': 'The metric threshold value'
            },
            'interval_between_workflows': {
              'default': 300,
              'description': 'Trigger workflow only if the last workflow was triggered earlier than interval-between-workflows seconds ago.\nif < 0  workflows can run concurrently.\n'
            }
          }
        },
        'cloudify.policies.types.threshold': {
          'source': 'file:///opt/manager/resources/cloudify/policies/threshold.clj',
          'properties': {
            'stability_time': {
              'default': 0,
              'description': 'How long a threshold must be breached before the triggers will be processed'
            },
            'policy_operates_on_group': {
              'default': False,
              'description': 'If the policy should maintain its state for the whole group\nor each node instance individually.\n'
            },
            'is_node_started_before_workflow': {
              'default': True,
              'description': 'Before triggering workflow, check if the node state is started'
            },
            'service': {
              'default': 'service',
              'description': 'The service name'
            },
            'upper_bound': {
              'default': True,
              'description': "boolean value for describing the semantics of the threshold.\nif 'true': metrics whose value is bigger than the threshold will cause the triggers to be processed.\nif 'false': metrics with values lower than the threshold will do so.\n"
            },
            'threshold': {
              'description': 'The metric threshold value'
            },
            'interval_between_workflows': {
              'default': 300,
              'description': 'Trigger workflow only if the last workflow was triggered earlier than interval-between-workflows seconds ago.\nif < 0  workflows can run concurrently.\n'
            }
          }
        }
      },
      'policy_triggers': {
        'cloudify.policies.triggers.execute_workflow': {
          'source': 'file:///opt/manager/resources/cloudify/triggers/execute_workflow.clj',
          'parameters': {
            'workflow_parameters': {
              'default': {
                
              },
              'description': 'Workflow paramters'
            },
            'force': {
              'default': False,
              'description': 'Should the workflow be executed even when another execution\nfor the same workflow is currently in progress\n'
            },
            'conn_timeout': {
              'default': 1000,
              'description': 'Connection timeout when making request to manager REST in ms'
            },
            'workflow': {
              'description': 'Workflow name to execute'
            },
            'socket_timeout': {
              'default': 1000,
              'description': 'Socket timeout when making request to manager REST in ms'
            },
            'allow_custom_parameters': {
              'default': False,
              'description': 'Should parameters not defined in the workflow parameters\nschema be accepted\n'
            }
          }
        }
      },
      'groups': {
        
      },
      'api_token': 'ojndfad24d57794c343db9371e9356daef325'
    }
  }
}
```



##Appendix A
File `hooks.yaml`
```yaml
hooks:
  - event_type: workflow_started
    implementation: cloudify-utilities-plugin.cloudify_rest.tasks.execute_as_workflow
    inputs:
      logger_file: /tmp/hooks_log.log
      prerender: true
      properties:
        hosts: ["insights-collector.eu01.nr-data.net"]
        port: 443
        ssl: true
        verify: true
      template_file: /opt/mgmtworker/config/rest.yaml
      params:
        account_id: "2675830"
        api_key: "NRII-ygXSaUb8ZBv8BVK5l7QlAGPC__yFjS9r"
    description: Sends to newrelic on workflow_started


  - event_type: workflow_succeeded
    implementation: cloudify-utilities-plugin.cloudify_rest.tasks.execute_as_workflow
    inputs:
      logger_file: /tmp/hooks_log.log
      prerender: true
      properties:
        hosts: ["insights-collector.eu01.nr-data.net"]
        port: 443
        ssl: true
        verify: true
      template_file: /opt/mgmtworker/config/rest.yaml
      params:
        account_id: "2675830"
        api_key: "NRII-ygXSaUb8ZBv8BVK5l7QlAGPC__yFjS9r"
    description: Sends to newrelic on workflow_succeeded


  - event_type: workflow_failed
    implementation: cloudify-utilities-plugin.cloudify_rest.tasks.execute_as_workflow
    inputs:
      logger_file: /tmp/hooks_log.log
      prerender: true
      properties:
        hosts: ["insights-collector.eu01.nr-data.net"]
        port: 443
        ssl: true
        verify: true
      template_file: /opt/mgmtworker/config/rest.yaml
      params:
        account_id: "2675830"
        api_key: "NRII-ygXSaUb8ZBv8BVK5l7QlAGPC__yFjS9r"
    description: Sends to newrelic on workflow_failed


  - event_type: workflow_cancelled
    implementation: cloudify-utilities-plugin.cloudify_rest.tasks.execute_as_workflow
    inputs:
      logger_file: /tmp/hooks_log.log
      prerender: true
      properties:
        hosts: ["insights-collector.eu01.nr-data.net"]
        port: 443
        ssl: true
        verify: true
      template_file: /opt/mgmtworker/config/rest.yaml
      params:
        account_id: "2675830"
        api_key: "NRII-ygXSaUb8ZBv8BVK5l7QlAGPC__yFjS9r"
    description: Sends to newrelic on workflow_cancelled


  - event_type: workflow_queued
    implementation: cloudify-utilities-plugin.cloudify_rest.tasks.execute_as_workflow
    inputs:
      logger_file: /tmp/hooks_log.log
      prerender: true
      properties:
        hosts: ["insights-collector.eu01.nr-data.net"]
        port: 443
        ssl: true
        verify: true
      template_file: /opt/mgmtworker/config/rest.yaml
      params:
        account_id: "2675830"
        api_key: "NRII-ygXSaUb8ZBv8BVK5l7QlAGPC__yFjS9r"
    description: Sends to newrelic on workflow_queued
```
File `/opt/mgmtworker/config/rest.yaml`
```yaml
rest_calls:
  - path: /v1/accounts/{{account_id}}/events
    method: POST
    headers:
      Content-type: application/json
      X-Insert-Key: {{api_key}}
    payload:
      - eventType: {{__inputs__.event_type|tojson}}
        deployment_id: {{__inputs__.deployment_id|tojson}}
        is_system_workflow: {{__inputs__.is_system_workflow|tojson}}
        blueprint_id: {{__inputs__.blueprint_id|tojson}}
        tenant_name: {{__inputs__.tenant_name|tojson}}
        workflow_id: {{__inputs__.workflow_id|tojson}}
        timestamp: {{__inputs__.timestamp|tojson}}
        execution_id: {{__inputs__.execution_id|tojson}}
    response_format: json
    recoverable_codes: []
    response_translation: {}
    response_expectation:
      - ['success', 'True']
```

