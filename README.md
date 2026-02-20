---
- name: Manage DSE Agent for Cassandra User
  hosts: dse_nodes
  vars:
    # Set this to the absolute path of your agent's bin directory
    agent_bin_path: "/home/cassandra/datastax-agent/bin/datastax-agent"
    # Action options: stop, start, restart
    action: restart

  tasks:
    - name: Find the PID of the Datastax Agent
      # Search for the jar file in the process list
      ansible.builtin.shell: "pgrep -u cassandra -f 'datastax-agent.*standalone.jar'"
      register: agent_pid
      failed_when: false
      changed_when: false

    - name: Kill the Agent process (SIGKILL)
      ansible.builtin.shell: "kill -9 {{ item }}"
      loop: "{{ agent_pid.stdout_lines }}"
      when: 
        - (action == 'stop' or action == 'restart')
        - agent_pid.stdout != ""
      become: yes
      become_user: cassandra

    - name: Start the Agent as Cassandra user
      # Use nohup and redirect output to ensure it stays running after Ansible exits
      ansible.builtin.shell: "nohup {{ agent_bin_path }} > /home/cassandra/agent_launch.log 2>&1 &"
      when: action == 'start' or action == 'restart'
      become: yes
      become_user: cassandra
      async: 10
      poll: 0

    - name: Verify the process is running
      ansible.builtin.shell: "pgrep -u cassandra -f 'datastax-agent.*standalone.jar'"
      register: verify_pid
      until: verify_pid.rc == 0
      retries: 5
      delay: 2
      when: action == 'start' or action == 'restart'
      changed_when: false
