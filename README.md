# checkpoint-checker1-status

Gaia Monitor - Enables the extraction of CheckPoint information from the default Gaia web interface to be index into Elastic Search for monitoring and alerting.

## Requirements

- Ansible Tower

- Ansible Module: checkpoint-management-server - https://github.com/worfinator/checkpoint-management-server

- CheckPoint Management Server account

- Firewall access enabled to the CheckPoint Management Server

````

## Example Playbook

Presuming that you are using this role within Ansible Tower with inventory groups named CHECKPOINT-Firewalls and CHECKPOINT-Management-Servers, Replace CP-01, CP-02 and CP-MS with valid host names if you wish to run this playbook standalone outside of Tower.

```yaml
- name: CheckPoint Blades Checker

  hosts: localhost

  vars:
    CheckPoints:
      - CP-01
      - CP-03
      - CP-02
    CheckPoint_Management_Servers:
      - CP-MS
    checkpoints_results: {}
    passed_devices: []
    failed_devices: []
    mgmt_task: status

  tasks:
    - name: Set hosts
      set_fact:
        CheckPoints: "{{ groups['CHECKPOINT-Firewalls'] }}"
        CheckPoint_Management_Servers: "{{ groups['CHECKPOINT-Management-Servers'] }}"
      when: groups['CHECKPOINT-Firewalls'] is defined and groups['CHECKPOINT-Management-Servers'] is defined

    - name: CheckPoints we will be querying
      debug:
        var: CheckPoints

    # talk to management server and get CP FW data
    - include_role: name=checkpoint-management-server

    - name: debug results
      debug:
        var: item
      with_items: "{{ checkpoints_results }}"

    # Check status of each FW
      - include_role: name=checkpoint-checker1-status
        with_items: "{{ checkpoints_results }}"
        loop_control:
          loop_var: checkpoint

      - name: set stat example
        set_stats:
          data:
            checkpoint_data: "{{ checkpoints_results }}"
          per_host: no

      - name: Output check results
        debug:
          var: checkpoints_results

      - name: Display info on any failed devices
        debug:
          msg:
            - "{{ failed_device }}"
            - " - Data check: {{ 'Ok' if checkpoints_results[failed_device].data_exists == True else 'Failed'}}"
            - " - IPS blade check: {{ checkpoints_results[failed_device].ips }}"
            - " - AntiBot blade check: {{ checkpoints_results[failed_device].ab }}"
            - " - AntiVirus blade check: {{ checkpoints_results[failed_device].av }}"
            - " {{ checkpoints_results[failed_device].message }}"
        with_items: "{{ failed_devices }}"
        loop_control:
          loop_var: failed_device

      - name: Fail check if issue with any CheckPoint device
        fail:
          msg:
            - "The Checker1 process failed the following {{ failed_devices|length }} devices"
            - "{{ failed_devices }}"
        when: failed_devices|length > 0
```

## License

GPL-2.0-or-later

## Author Information

For feedback and comments please contact me via:

mark.baldwin.nz@gmail.com
````
