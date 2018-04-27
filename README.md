# Ansible Role: Google Compute Engine

Configure Google Compute Engine

Cloud Provider:
```YAML
cloud_provider:
  provider: "gce"
  region: "europe-west1"
  zone: "europe-west1-d"
  project: "darkbulb-lab"
```

Cloud Compute:
```YAML
cloud_compute:
# Instance Information
  instance:
  # Entry
    jenkins-000:
      name: jenkins-000
      # The Cloud Size
      size: n1-standard-1
      # The Cloud Image
      image: centos-7
      credential:
        username: victorock
        key: jenkins
      # Instance Network
      network:
        subnet: jenkins
        public: yes
      # Instance Tags
      tags:
        - jenkins
```

Cloud Network:
```YAML
cloud_network:
  cidr: "10.1.0.0/16"

# DNS Information
  dns:
    # Entry
    jenkins:
      name: jenkins
      record: jenkins.devops.victorock.io.
      zone: devops.victorock.io.

# Firewall Information
  firewall:
    # Entry
    jenkins:
      name: jenkins
      subnet: jenkins
      rules:
        - name: "jenkins-ssh-all"
          src_cidr: 0.0.0.0/0
          dst_port: 22
          proto: tcp

        - name: "jenkins-http-all"
          src_cidr: 0.0.0.0/0
          dst_port: 8080
          proto: tcp

        - name: "jenkins-https-all"
          subnet: jenkins
          src_cidr: 0.0.0.0/0
          dst_port: 8443
          proto: tcp

        - name: "jenkins-icmp-all"
          subnet: jenkins
          src_cidr: 0.0.0.0/0
          proto: icmp

# Loadbalance Information
  loadbalance:
    # Entry
    jenkins-http:
      name: jenkins-http
      # Reference to Public Entry
      vip: jenkins
      protocol: tcp
      port: 80
      members:
        # Reference to Instance Entry
        - jenkins-000
      healthcheck:
        name: jenkins-http-8080
        type: HTTP
        port: 8080
        protocol: tcp
        url: "/"

# Public Information
  public:
    # Entry
    jenkins:
      name: jenkins

# Subnet Information
  subnet:
    # Entry
    jenkins:
      name: jenkins
      # CIDR: 10.1.*1*.0/*24*
      cidr: "{{ cloud_provider.cidr | ipsubnet(24, 1) }}"
```

## Example

Playbook: Instance Provisioning

```YAML
---
- name: "Provision: Google Compute Engine"
  hosts: localhost
  connection: local
  gather_facts: false
  become: false

  tasks:
    - include_role:
        name: victorock.gce_config
        tasks_from: compute
      vars:
        cloud_compute:
          instance:
            jenkins-000:
              name: jenkins-000
              size: n1-standard-1
              image: centos-7

```

## License

MIT / BSD

## Author Information
