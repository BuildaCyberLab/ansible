# Ansible

![image](https://github.com/user-attachments/assets/23a5ae1c-273e-4131-8796-4e9f7b7dba04)

- [Ansible Official Website](https://www.ansible.com)  
- [Ansible Documentation](https://docs.ansible.com)  
- [Getting Started with Ansible](https://docs.ansible.com/ansible/latest/getting_started/index.html)  
- [Ansible Community Forum](https://forum.ansible.com)  
- [Ansible GitHub Repository](https://github.com/ansible/ansible)

Ansible is a powerful tool for network automation. It uses YAML playbooks to define tasks and automates complex configurations. Here’s what you need to know to get started effectively:

---

### **1. Key Concepts**
- **Inventory:** List of managed nodes (devices or servers).
- **Playbooks:** YAML scripts to define automation tasks.
- **Modules:** Pre-built tasks for networks (e.g., `ios_config` for Cisco).
- **Connection Types:** Use `network_cli` for network devices.

---

### **2. Setup Basics**
1. Install Ansible:
   ```bash
   sudo apt update
   sudo apt install ansible
   ```

2. Install dependencies for network modules:
   ```bash
   ansible-galaxy collection install ansible.netcommon
   ansible-galaxy collection install cisco.ios
   ```

3. Configure inventory (`inventory.yml`):
   ```yaml
   all:
     hosts:
       router1:
         ansible_host: 192.168.1.1
         ansible_user: admin
         ansible_password: password
         ansible_network_os: ios
   ```

---

### **3. Basic Playbook Example**
Create a playbook (`configure_router.yml`) to set a hostname on a Cisco router:
```yaml
- name: Configure Cisco Router
  hosts: all
  gather_facts: no
  tasks:
    - name: Set hostname
      cisco.ios.ios_config:
        lines:
          - hostname Router1
```

Run the playbook:
```bash
ansible-playbook -i inventory.yml configure_router.yml
```

---

### **4. Common Tasks**
#### Backup Configurations:
```yaml
- name: Backup running-config
  hosts: all
  gather_facts: no
  tasks:
    - name: Save config
      cisco.ios.ios_config:
        backup: yes
        src: running-config
```

#### Push ACLs:
```yaml
- name: Configure ACLs
  hosts: all
  gather_facts: no
  tasks:
    - name: Apply ACL
      cisco.ios.ios_config:
        lines:
          - ip access-list standard 10
          - permit 192.168.1.0 0.0.0.255
```

---

### **5. Debugging Tips**
- Test connections:
  ```bash
  ansible -i inventory.yml all -m ping
  ```
- Use `--check` to test changes without applying:
  ```bash
  ansible-playbook -i inventory.yml configure_router.yml --check
  ```
- Add `-v` for verbose output.

---

### **1. Key Concepts in Ansible**
Understanding the core elements of Ansible is essential to mastering it for network automation. Let’s break down each concept with practical clarity.

---

#### **1.1 Inventory**
- The **inventory** is where you define the devices or systems Ansible will manage. 
- It can be a simple text file (`.ini` format) or written in YAML for more structure.

**Example: Basic Inventory (INI)**
```ini
[routers]
router1 ansible_host=192.168.1.1 ansible_user=admin ansible_password=password ansible_network_os=ios
router2 ansible_host=192.168.1.2 ansible_user=admin ansible_password=password ansible_network_os=ios
```

**Example: Structured Inventory (YAML)**
```yaml
all:
  hosts:
    router1:
      ansible_host: 192.168.1.1
      ansible_user: admin
      ansible_password: password
      ansible_network_os: ios
    router2:
      ansible_host: 192.168.1.2
      ansible_user: admin
      ansible_password: password
      ansible_network_os: ios
```

- **Groups:** You can group devices for easier management. For example:
```yaml
routers:
  hosts:
    router1:
    router2:
```

---

#### **1.2 Playbooks**
- Playbooks are the heart of Ansible. They define the tasks you want to automate, written in **YAML** (a simple, human-readable data format).

**Key Structure of a Playbook:**
1. **`hosts:`** Targets the inventory group or device.
2. **`tasks:`** A list of actions to perform.
3. **Modules:** Each task uses a specific Ansible module to perform an operation (e.g., `ios_config` for network configuration).

**Example Playbook:**
```yaml
- name: Configure a router
  hosts: routers
  gather_facts: no
  tasks:
    - name: Set hostname
      cisco.ios.ios_config:
        lines:
          - hostname Router1
```

- **YAML Syntax Rules:**
  - Always use spaces, not tabs.
  - Indentation is critical; improper spacing breaks the playbook.

---

#### **1.3 Modules**
Modules are pre-built scripts in Ansible that perform specific tasks. For network automation, Ansible provides modules through collections like `cisco.ios`.

**Examples of Network Modules:**
- **`ios_config`:** Configures Cisco IOS devices.
- **`nxos_config`:** Configures Cisco Nexus devices.
- **`eos_config`:** Configures Arista devices.

**How Modules Work:**
- You provide input parameters (like commands or config lines), and the module does the work.

**Example with `ios_config`:**
```yaml
- name: Configure VLAN
  hosts: routers
  tasks:
    - name: Create VLAN 10
      cisco.ios.ios_config:
        lines:
          - vlan 10
          - name Sales
```

---

#### **1.4 Connection Types**
Since network devices don’t have an operating system like servers, you must use specific connection methods:

- **`network_cli`:** Default for SSH-based network device management.
- **`httpapi`:** For devices with REST APIs (e.g., Cisco IOS-XE).

**Set Connection in Inventory:**
```yaml
router1:
  ansible_host: 192.168.1.1
  ansible_connection: network_cli
```

---

#### **1.5 Facts**
- Facts are details Ansible collects about a system (e.g., OS version, IP address).
- For network devices, facts are disabled by default because they can slow down execution.

**Disabling Facts:**
```yaml
gather_facts: no
```

**Enable Specific Facts Collection (Optional):**
```yaml
tasks:
  - name: Gather facts
    cisco.ios.ios_facts:
```

---

### **Why These Concepts Matter**
1. **Inventory** defines **what** you control.
2. **Playbooks** define **how** you control it.
3. **Modules** execute **specific actions** on the devices.
4. **Connection Types** ensure Ansible speaks the right language for the device.

---

### **2. Setup Basics: Getting Started with Ansible**

To use Ansible for network automation, you must prepare your environment properly. This involves installing Ansible, setting up the necessary collections and dependencies, and configuring your inventory. Let’s go step-by-step for a full understanding.

---

### **2.1 Installing Ansible**
Ansible runs on a control machine (your computer or server). It communicates with managed devices using SSH or APIs. Here’s how to install it:

#### **Steps for Installation**
1. **Update Your System:**
   Ensure your system is up to date.
   ```bash
   sudo apt update
   sudo apt upgrade
   ```

2. **Install Ansible:**
   Install Ansible from the package manager or via Python’s `pip`.
   ```bash
   sudo apt install ansible -y
   ```

3. **Verify Installation:**
   Check if Ansible is installed.
   ```bash
   ansible --version
   ```
   You should see the installed version and its dependencies.

#### **Why Install via `pip` Sometimes?**
Some environments require the latest version of Ansible, which might not be available in the system's package manager. Use `pip`:
```bash
pip install ansible
```

---

### **2.2 Install Network Collections**
Ansible organizes specialized modules for networks into **collections**. These collections contain modules for configuring Cisco, Arista, Juniper, or other devices.

#### **Install Required Collections**
1. **Install `ansible.netcommon`:**
   This collection provides foundational tools for network automation.
   ```bash
   ansible-galaxy collection install ansible.netcommon
   ```

2. **Install Vendor-Specific Collections:**
   For example, to work with Cisco IOS devices:
   ```bash
   ansible-galaxy collection install cisco.ios
   ```

#### **Verify Collection Installation:**
List installed collections:
```bash
ansible-galaxy collection list
```
You should see entries like `ansible.netcommon` and `cisco.ios`.

---

### **2.3 Inventory Configuration**
The **inventory** is a file that tells Ansible which devices to manage and how to connect to them. This step ensures Ansible knows the devices, usernames, passwords, and operating systems.

#### **Creating an Inventory File**
An inventory can be in two formats:
1. **INI Format:**
   A simple list with connection details:
   ```ini
   [routers]
   router1 ansible_host=192.168.1.1 ansible_user=admin ansible_password=password ansible_network_os=ios
   router2 ansible_host=192.168.1.2 ansible_user=admin ansible_password=password ansible_network_os=ios
   ```

2. **YAML Format:**
   A structured format offering more flexibility:
   ```yaml
   all:
     hosts:
       router1:
         ansible_host: 192.168.1.1
         ansible_user: admin
         ansible_password: password
         ansible_network_os: ios
       router2:
         ansible_host: 192.168.1.2
         ansible_user: admin
         ansible_password: password
         ansible_network_os: ios
   ```

#### **Key Inventory Parameters**
- `ansible_host`: IP address or hostname of the device.
- `ansible_user`: Username for SSH or API login.
- `ansible_password`: Password for SSH or API login.
- `ansible_network_os`: Device's network OS (e.g., `ios` for Cisco routers).

---

### **2.4 Testing the Setup**
After setting up Ansible and the inventory, test connectivity to ensure everything works.

#### **Ping Test:**
Ansible doesn’t use the traditional `ping` command for devices. Instead, it checks SSH connectivity using a "ping module."
```bash
ansible -i inventory.yml all -m ping
```
- **Success:** You’ll see a green "SUCCESS" message.
- **Failure:** Check your inventory details and ensure SSH is enabled on the devices.

#### **Gather Facts:**
To collect basic information about a device, use the `gather_facts` module:
```bash
ansible -i inventory.yml all -m gather_facts
```

---

### **2.5 Configure Playbook Directory**
Organize your files to keep projects clean and manageable.

**Example Directory Structure:**
```plaintext
network-automation/
├── inventory.yml
├── playbooks/
│   ├── configure_router.yml
│   ├── backup_config.yml
├── group_vars/
│   ├── all.yml
```

- **`inventory.yml`:** Defines your managed devices.
- **`playbooks/`:** Stores YAML playbooks for automation tasks.
- **`group_vars/`:** Stores variables for device groups (e.g., credentials).

---

### **2.6 Running Playbooks**
To run a playbook, use the `ansible-playbook` command:
```bash
ansible-playbook -i inventory.yml playbooks/configure_router.yml
```

#### **Key Options:**
- `-i inventory.yml`: Specifies the inventory file.
- `--check`: Runs the playbook in "dry run" mode (no changes made).
- `-v`: Increases verbosity for debugging.

---

### **Final Setup Checklist**
1. Install Ansible and dependencies.
2. Install required network collections.
3. Create and test your inventory file.
4. Organize playbooks and variables.
5. Run a playbook to verify functionality.

### **3. Basic Playbook Example - Bird’s Eye View**
To deeply understand how Ansible operates in a real-world scenario, let’s walk through the **playbook example** from a high-level perspective, breaking down every component and explaining the logic.

---

#### **Scenario: Configuring a Router’s Hostname**

**Problem:**  
You need to automate setting the hostname of a Cisco router. Manually logging into each router via SSH and typing commands is time-consuming and error-prone.

**Solution:**  
Use an Ansible playbook to:
1. Connect to the router.
2. Execute the configuration commands.
3. Confirm success.

---

### **Bird’s Eye View: How the Playbook Fits Together**
Here’s the playbook for reference:

```yaml
- name: Configure Cisco Router
  hosts: all
  gather_facts: no
  tasks:
    - name: Set hostname
      cisco.ios.ios_config:
        lines:
          - hostname Router1
```

#### **3.1. The Playbook Components**

1. **`- name:`**  
   This is a human-readable description of the play.  
   Example: `"Configure Cisco Router"` explains the purpose of this playbook.

2. **`hosts:`**  
   Specifies the devices (from the inventory) this playbook targets.  
   Example: `all` means all devices in the inventory file.

3. **`gather_facts:`**  
   By default, Ansible collects facts (details about the system).  
   - For network devices, gathering facts slows execution and is often unnecessary.
   - Example: `gather_facts: no` skips this step, speeding up the process.

4. **`tasks:`**  
   A list of actions to execute on the target devices.

5. **`- name:`** (within `tasks`)  
   A human-readable label for the task.  
   Example: `"Set hostname"` explains what this specific task will do.

6. **`cisco.ios.ios_config:`**  
   This is the **module** being used.  
   - `ios_config` sends configuration commands to Cisco IOS devices.
   - Ansible uses this to apply the new hostname.

7. **`lines:`**  
   The actual configuration commands to send.  
   Example:  
   ```yaml
   lines:
     - hostname Router1
   ```
   This command changes the router’s hostname to "Router1".

---

#### **3.2. Steps in Execution**

When you run the playbook, here’s what happens:

1. **Step 1: Read the Inventory**  
   - Ansible reads the `inventory.yml` file to determine the devices to connect to.
   - Example Inventory:
     ```yaml
     all:
       hosts:
         router1:
           ansible_host: 192.168.1.1
           ansible_user: admin
           ansible_password: password
           ansible_network_os: ios
     ```

2. **Step 2: Authenticate and Connect**  
   - Ansible uses `network_cli` (default for network devices) to SSH into the router.
   - Credentials (`ansible_user` and `ansible_password`) are pulled from the inventory.

3. **Step 3: Execute the Task**  
   - The `ios_config` module sends the command `hostname Router1` to the router.
   - Ansible ensures the command is executed as expected:
     - If the current hostname is already "Router1," it skips making changes (idempotency).
     - If the hostname is different, it applies the change.

4. **Step 4: Verify Success**  
   - Ansible checks the response from the router to confirm the command succeeded.
   - If there’s an error (e.g., bad credentials), it reports the failure.

---

#### **3.3. Key Features in Action**

1. **Automation and Consistency**  
   The playbook ensures every router gets the exact same configuration without human error.

2. **Idempotency**  
   - If the desired state (hostname = "Router1") already exists, Ansible does nothing.
   - This ensures no unnecessary changes are made.

3. **Scalability**  
   - By targeting `all` devices, the playbook can configure multiple routers at once.
   - You can add more devices to the inventory without changing the playbook.

---

### **Why This Playbook is Powerful**

1. **Reusable:**  
   This playbook can be modified to configure other settings (e.g., VLANs, IP routing).

2. **Efficient:**  
   A single playbook replaces manual SSH logins and error-prone CLI commands.

3. **Clear:**  
   The YAML structure is simple and easy to read, even for beginners.

4. **Error-Handling Built-In:**  
   Ansible handles connection issues, command validation, and ensures the task completes safely.

---

### **How to Run This Playbook**

1. Save the playbook as `configure_router.yml`.  
2. Create an inventory file (`inventory.yml`) with the target devices.  
3. Run the playbook using the following command:
   ```bash
   ansible-playbook -i inventory.yml configure_router.yml
   ```

4. Verify the results:
   - Log into the router manually to confirm the hostname has changed.
   - Use verbose mode (`-v`) for more detailed logs:
     ```bash
     ansible-playbook -i inventory.yml configure_router.yml -v
     ```

---

### **Expanding the Scenario**

**Example: Configuring Multiple Routers**  
Add more devices to the inventory:
```yaml
all:
  hosts:
    router1:
      ansible_host: 192.168.1.1
      ansible_user: admin
      ansible_password: password
      ansible_network_os: ios
    router2:
      ansible_host: 192.168.1.2
      ansible_user: admin
      ansible_password: password
      ansible_network_os: ios
```

The same playbook will now configure both routers automatically.

---

### **Final Takeaway**
This playbook simplifies network automation by removing repetitive manual steps, ensuring consistency, and scaling easily for multiple devices. It’s a foundation you can build on for larger and more complex network tasks.


---

### **4. Common Tasks in Ansible for Network Automation**
To automate networks effectively, you need a solid understanding of the most common tasks Ansible can perform. These include **backing up configurations**, **pushing configurations**, **managing access control lists (ACLs)**, and more. Below is a breakdown of how to perform these tasks, why they matter, and the underlying mechanics.

---

### **4.1 Backup Configurations**
Backing up device configurations is critical for disaster recovery and tracking changes.

#### **Why This Task Is Important:**
- Prevent data loss.
- Compare configurations over time.
- Quickly restore a device to a known working state.

#### **How It Works:**
The `ios_config` module allows you to back up the running or startup configurations of Cisco devices. When the `backup: yes` parameter is used, Ansible saves a local backup file.

#### **Playbook Example:**
```yaml
- name: Backup Device Configuration
  hosts: all
  gather_facts: no
  tasks:
    - name: Backup running configuration
      cisco.ios.ios_config:
        backup: yes
```

- **Output:** The configuration will be saved in Ansible’s working directory under a timestamped filename.

#### **Key Considerations:**
- Ensure you have write permissions in the directory where backups are saved.
- Automate backups on a schedule using tools like **cron** or a CI/CD pipeline.

---

### **4.2 Pushing Configurations**
Automating configuration changes reduces manual effort and human error.

#### **Why This Task Is Important:**
- Apply consistent configurations across multiple devices.
- Automate routine changes (e.g., VLAN updates, interface configurations).
- Enforce standard network policies.

#### **How It Works:**
The `ios_config` module pushes configuration commands to devices. You define the commands using the `lines` parameter.

#### **Playbook Example:**
**Push a hostname change:**
```yaml
- name: Set Hostname
  hosts: all
  gather_facts: no
  tasks:
    - name: Configure hostname
      cisco.ios.ios_config:
        lines:
          - hostname NewHostname
```

**Push multiple commands at once:**
```yaml
- name: Configure Interfaces
  hosts: all
  gather_facts: no
  tasks:
    - name: Set interface descriptions
      cisco.ios.ios_config:
        lines:
          - interface GigabitEthernet0/1
          - description Link to Core
          - interface GigabitEthernet0/2
          - description Link to Access
```

#### **Key Considerations:**
- Use the `--check` flag to simulate changes before applying them.
- Maintain a central repository of configuration templates for consistency.

---

### **4.3 Managing ACLs**
Access control lists (ACLs) are essential for controlling traffic and enhancing security.

#### **Why This Task Is Important:**
- Control access to network resources.
- Filter unwanted traffic.
- Enforce security policies.

#### **How It Works:**
The `ios_config` module can create or update ACLs. You define ACL rules as a list of lines.

#### **Playbook Example:**
**Apply a Standard ACL:**
```yaml
- name: Configure Standard ACL
  hosts: all
  gather_facts: no
  tasks:
    - name: Add standard ACL
      cisco.ios.ios_config:
        lines:
          - ip access-list standard 10
          - permit 192.168.1.0 0.0.0.255
          - deny any
```

**Apply an Extended ACL:**
```yaml
- name: Configure Extended ACL
  hosts: all
  gather_facts: no
  tasks:
    - name: Add extended ACL
      cisco.ios.ios_config:
        lines:
          - ip access-list extended 100
          - permit tcp 192.168.1.0 0.0.0.255 any eq 80
          - deny ip any any
```

#### **Key Considerations:**
- Ensure ACLs are applied to the correct interfaces.
- Use Ansible’s `--diff` flag to see changes before and after applying ACLs.

---

### **4.4 Gathering Facts**
Facts provide critical information about a device, like its OS version, uptime, and interfaces.

#### **Why This Task Is Important:**
- Understand the current state of a device before making changes.
- Automate decisions based on device-specific information (e.g., different commands for different OS versions).

#### **How It Works:**
The `ios_facts` module collects detailed information about a device.

#### **Playbook Example:**
```yaml
- name: Gather Device Facts
  hosts: all
  gather_facts: no
  tasks:
    - name: Retrieve facts
      cisco.ios.ios_facts:
```

**Facts Collected Include:**
- OS version
- Serial number
- Interface details
- Running configuration

#### **Key Considerations:**
- Gathering facts can slow down execution; only use it when necessary.
- Use filters to extract specific facts.

---

### **4.5 Verifying and Testing Configurations**
Before applying changes, always verify the configuration.

#### **Why This Task Is Important:**
- Prevent misconfigurations that could disrupt operations.
- Test changes without impacting live traffic.

#### **How It Works:**
Use the `--check` flag or the `diff` parameter to preview changes.

#### **Example: Dry Run with `--check`:**
```bash
ansible-playbook -i inventory.yml configure_router.yml --check
```

#### **Example: Preview Changes with `diff`:**
```yaml
- name: Configure VLAN with Diff
  hosts: all
  gather_facts: no
  tasks:
    - name: Add VLAN with diff enabled
      cisco.ios.ios_config:
        lines:
          - vlan 20
          - name Engineering
        diff: true
```

---

### **4.6 Advanced Example: Full Workflow**
Combine tasks for a complete workflow:
1. Backup the current configuration.
2. Push new configurations.
3. Verify the changes.

**Playbook:**
```yaml
- name: Full Workflow: Backup and Configure
  hosts: all
  gather_facts: no
  tasks:
    - name: Backup Configuration
      cisco.ios.ios_config:
        backup: yes

    - name: Configure VLAN
      cisco.ios.ios_config:
        lines:
          - vlan 10
          - name Sales

    - name: Verify Changes
      cisco.ios.ios_config:
        lines:
          - show vlan brief
        register: vlan_output

    - name: Display VLAN Output
      debug:
        var: vlan_output
```

---

### **Summary of Key Best Practices**
1. **Backups First:** Always back up configurations before making changes.
2. **Test Changes:** Use `--check` and `--diff` to validate changes before applying them.
3. **Keep It Modular:** Write small, reusable playbooks for specific tasks.
4. **Use Templates:** Store common configurations in templates for consistency.
5. **Secure Inventory:** Protect sensitive information like passwords using **Ansible Vault**.

### **5. Debugging Tips in Ansible**

Debugging is critical when using Ansible. Even small mistakes in configurations, playbooks, or device setups can cause errors. To solve these issues effectively, you need to understand how to find the root cause and fix it.

---

### **5.1 Test Connections with `ping` Module**
Before running playbooks, always confirm Ansible can reach your devices.

**Command to Test Connection:**
```bash
ansible -i inventory.yml all -m ping
```

- **What it does:**
  - `-i inventory.yml`: Specifies the inventory file.
  - `all`: Targets all devices in the inventory.
  - `-m ping`: Uses the `ping` module to verify connectivity.

- **What to Expect:**
  - Success:
    ```yaml
    router1 | SUCCESS => {
      "changed": false,
      "ping": "pong"
    }
    ```
  - Failure (Example Causes):
    - Wrong credentials → Check `ansible_user` and `ansible_password`.
    - Incorrect IP/hostname → Verify `ansible_host`.
    - SSH blocked or misconfigured → Ensure SSH is enabled on the device.

---

### **5.2 Use `--check` Mode**
The `--check` flag simulates running a playbook without making changes. It’s like a "dry run" to confirm your playbook is correct.

**Command:**
```bash
ansible-playbook -i inventory.yml playbook.yml --check
```

- **What it does:**
  - Shows the tasks Ansible would perform.
  - Helps identify syntax errors or misconfigurations without affecting devices.

- **Example Output:**
  ```bash
  TASK [Set hostname] ******************************************************
  ok: [router1]
  ```

If there’s an issue, Ansible shows an error message instead.

---

### **5.3 Increase Verbosity**
Add `-v`, `-vv`, or `-vvv` to increase verbosity and get detailed output about what Ansible is doing.

**Example Command:**
```bash
ansible-playbook -i inventory.yml playbook.yml -vvv
```

- **What it does:**
  - `-v`: Basic details of what’s happening.
  - `-vv`: More context about each step.
  - `-vvv`: Full debugging details (e.g., SSH commands, responses).

- **Use Case:**
  When a task fails, verbosity shows the raw error message from the device, helping you pinpoint the issue.

---

### **5.4 Debug Module**
The `debug` module prints variables or messages, which is helpful for troubleshooting playbooks.

**Example: Print a Variable**
```yaml
- name: Debug Example
  hosts: all
  tasks:
    - name: Display hostname
      debug:
        var: inventory_hostname
```

- **Output:**
  ```yaml
  TASK [Display hostname] ************************************************
  ok: [router1] => {
      "inventory_hostname": "router1"
  }
  ```

**Example: Custom Message**
```yaml
- name: Display Custom Message
  hosts: all
  tasks:
    - name: Show Message
      debug:
        msg: "Connecting to {{ inventory_hostname }}"
```

- **Output:**
  ```yaml
  TASK [Show Message] ****************************************************
  ok: [router1] => {
      "msg": "Connecting to router1"
  }
  ```

---

### **5.5 Check Syntax Before Running**
Use the `--syntax-check` flag to ensure your playbook syntax is correct.

**Command:**
```bash
ansible-playbook playbook.yml --syntax-check
```

- **What it does:**
  - Verifies YAML formatting.
  - Detects indentation issues or missing fields.
  - Doesn’t run the playbook.

- **Example Output:**
  ```bash
  playbook: playbook.yml
  ```

---

### **5.6 Handle Module Errors**
When a task fails, Ansible provides a detailed error message. Key steps to handle it:

1. **Read the Error Message:**
   - Example:
     ```yaml
     TASK [Set hostname] ******************************************************
     fatal: [router1]: FAILED! => {
       "msg": "Authentication failed."
     }
     ```

2. **Common Fixes:**
   - **Authentication Failed:** Check `ansible_user` and `ansible_password`.
   - **Timeout:** Verify `ansible_host` or network connectivity.
   - **Syntax Error in Command:** Check your `lines` or `commands` in the module.

3. **Re-run Specific Task:**
   Use `--start-at-task` to re-run only the failing task:
   ```bash
   ansible-playbook playbook.yml --start-at-task="Set hostname"
   ```

---

### **5.7 Review Logs**
If verbosity doesn’t provide enough detail, review logs for deeper insight. Enable logging in `ansible.cfg`.

**Edit `ansible.cfg`:**
```ini
[defaults]
log_path = ansible.log
```

- **What it does:**
  - Creates a detailed log of all playbook runs.
  - Logs every command, response, and error.

- **Inspect Logs:**
  Open `ansible.log` to review all actions and find the root cause of issues.

---

### **5.8 Common Debugging Techniques**
1. **Divide and Conquer:**
   - Test small playbooks with one task at a time.
   - Add complexity after verifying each task.

2. **Run Ad-Hoc Commands:**
   Use `ansible` commands directly to test specific modules or commands.
   ```bash
   ansible -i inventory.yml router1 -m cisco.ios.ios_facts
   ```

3. **Use Default Values:**
   Ensure required variables are set by providing defaults:
   ```yaml
   vars:
     hostname: "{{ hostname | default('Router1') }}"
   ```

---

### **Summary**
Debugging with Ansible requires a systematic approach:
1. Verify connectivity (`ping` module).
2. Test playbooks safely (`--check`).
3. Enable verbosity for detailed output (`-vvv`).
4. Use the `debug` module to inspect variables and messages.
5. Review syntax before running (`--syntax-check`).
6. Log every action with `ansible.cfg`.

Always isolate issues and test in small steps. Master debugging, and Ansible becomes your most reliable tool.
