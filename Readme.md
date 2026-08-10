# Course Material:
Course Name: Ansible Basics: Automation Technical Overview (DO007) - By Red Hat
Course link: https://rhtapps.redhat.com/promo/course/do007
Course Content link: https://github.com/rht-labs/do007

An **Ansible Playbook** is essentially an instruction manual for your automation.

In technical terms, it is a blueprint of automation tasks—which are complex IT actions executed with limited or no human involvement.

Here are the key things to know about playbooks:

* **Format:** They are written in YAML (Yet Another Markup Language), which makes them easy for humans to read and write.
* **Declarative:** Instead of writing scripts detailing *how* to do something, playbooks describe the *desired end state* of your systems. Ansible automatically figures out how to get the systems to that state.
* **Structure:** A playbook contains one or more "plays". Each play maps a group of hosts (servers) to well-defined roles or tasks. For example, one play might configure web servers, and another might configure database servers.
* **Purpose:** They are used to manage configurations, deploy applications, orchestrate complex multi-tier rollouts, or perform any other IT automation task.

*Think of it like a recipe: the playbook contains the list of ingredients (variables/settings) and the step-by-step instructions (tasks) needed to bake the cake (configure your servers).*



# What is Ansible Automation Platform (AAP)?

**Red Hat Ansible Automation Platform (AAP)** is an enterprise-grade automation solution designed to manage IT infrastructure across servers, cloud platforms, and network devices using the power of Ansible. In simple terms, it's a complete, stable, and scalable toolkit built on top of the open-source Ansible engine to handle complex automation reliably in a business environment.

Here are the key things to know about AAP:

* **What It Does:** It bridges the gap between developers and operations teams by providing a standardized way to automate tasks like application deployment, configuration management, security patching, and orchestration (coordinating complex workflows across many systems).
* **The Engine:** At its core, AAP uses **Ansible**, which is known for its simplicity. Ansible uses **Playbooks** (written in YAML) to describe the desired state of the system. It uses SSH for Linux/Unix and PowerShell remoting for Windows, meaning there's no need to install any special software (agents) on the target machines.
* **Key Components:**
    * **Automation Controller (formerly Ansible Tower):** A web-based user interface that allows you to manage Ansible automation with visual dashboards and detailed logging.
    * **Automation Hub:** A central repository for Ansible content, allowing teams to share and reuse automation components (like collections and roles).
    * **Execution Environments:** Containerized environments (based on Docker or Podman) that ensure consistency and security by standardizing the tools and dependencies used during automation runs.
* **Purpose:** It is used by companies to increase operational efficiency, reduce human error, speed up application delivery, and enforce consistent configurations across their entire infrastructure.

**Think of it like this:** If Ansible is a powerful, flexible wrench that a skilled mechanic can use to fix anything, **Ansible Automation Platform** is the entire high-tech toolbox that includes a vise to hold the parts steady, a torque wrench to ensure perfect tightness, and a detailed repair manual, all wrapped in a secure, easy-to-use package for the entire garage.

# Ansible Facts (or Gathered Facts):

Ansible Facts are simply built-in details that Ansible automatically collects about the computers or servers you are managing. Think of it as Ansible doing a quick background check on the machine before it starts doing any work on it.

For example, it automatically finds out things like:
* What operating system it is running (like Ubuntu, CentOS, or Windows)
* Its IP addresses (how to reach it on the network)
* How much memory (RAM) it has available
* How much storage space is on the hard drive
* What kind of processor it has
* And much more...

# Ansible Constructs:

* **Conditionals:** Rules that decide if a task should run.
  * *Example:* Only install a package if the operating system is Ubuntu.
    ```yaml
    when: ansible_os_family == "Debian"
    ```
* **Loops:** Used to repeat a single task multiple times with different items.
  * *Example:* Creating multiple user accounts using just one task.
    ```yaml
    loop:
      - alice
      - bob
    ```
* **Variables:** Placeholders for values that might change or need to be reused.
  * *Example:* Setting a port number variable.
    ```yaml
    vars:
      port_number: 8080
    ```
* **Tasks:** The individual, step-by-step actions Ansible performs.
  * *Example:* Installing a package.
    ```yaml
    tasks:
      - name: Install nginx
        ansible.builtin.package:
          name: nginx
          state: present
    ```
* **Handlers:** Special tasks that only run when triggered (notified) by a change made by another task.
  * *Example:* Restarting a service only when notified.
    ```yaml
    handlers:
      - name: Restart nginx
        ansible.builtin.service:
          name: nginx
          state: restarted
    ```
* **Roles:** A way to bundle tasks, variables, and files together into a structured, reusable package.
  * *Example:* Using a database role in a playbook.
    ```yaml
    roles:
      - role: my_database_role
    ```
* **Collections:** A larger distribution format that bundles modules, roles, and plugins together, often provided by vendors.
  * *Example:* Using a module from the AWS collection.
    ```yaml
    - amazon.aws.ec2_instance:
        name: my-server
    ```
* **Inventory:** The list or database of servers (hosts) that Ansible manages.
  * *Example:* Grouping servers in an inventory file.
    ```ini
    [webservers]
    web1.example.com
    web2.example.com
    ```
* **Modules:** The actual tools (small programs) Ansible uses in the background to do the work.
  * *Example:* Using the copy module.
    ```yaml
    ansible.builtin.copy:
      src: localfile.txt
      dest: /remote/path/file.txt
    ```
* **Templates:** Files (usually written in Jinja2 format) that can have variables injected into them before being placed on a server.
  * *Example:* Using the template module to deploy a Jinja2 file.
    ```yaml
    ansible.builtin.template:
      src: config.j2
      dest: /etc/config.conf
    ```
* **Playbooks:** The main YAML files that orchestrate all the tasks, tying everything else together.
  * *Example:* A basic playbook structure.
    ```yaml
    - name: Configure Web Servers
      hosts: webservers
      tasks:
        - name: Ensure nginx is running
          ansible.builtin.service:
            name: nginx
            state: started
    ```
* **Facts:** Information Ansible automatically gathers about the remote systems before running tasks.
  * *Example:* Using a fact in a task or message.
    ```yaml
    - name: Print OS version
      ansible.builtin.debug:
        msg: "The OS is {{ ansible_distribution }}"
    ```

# Ansible Handlers in Detail:

**Handlers** are special tasks in Ansible that only run when they are explicitly "notified" by another task. They are almost exclusively used to restart services or reload configurations only when an actual change has occurred.

### How they work:
1. **The Trigger (`notify`):** A regular task does some work (like copying a new configuration file). If that task actually makes a change on the server, it triggers a `notify` command.
2. **The Action (The Handler):** The handler sits patiently in a separate section. It will only execute at the very end of the play, and *only* if it received a notification. If the regular task didn't change anything (e.g., the config file was already up-to-date), the handler does nothing.

### Why use Handlers?
**Efficiency and Idempotency.** You don't want to restart your database or web server every single time you run a playbook if the settings haven't changed. Handlers ensure that disruptive actions (like service restarts) only happen when absolutely necessary.

### Example:
```yaml
tasks:
  - name: Update Nginx configuration file
    ansible.builtin.copy:
      src: new_nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart Nginx # This triggers the handler ONLY if the file was actually changed

handlers:
  - name: Restart Nginx # The name here must exactly match the 'notify' directive above
    ansible.builtin.service:
      name: nginx
      state: restarted
```

## Templates: 

* Templates are files that dynamically generate content by using variables, conditionals, and loops and filters.
* Templates are files that can have variables injected into them before being placed on a server. They are used to create dynamic configuration files. 

For example, you can create a template for a configuration file that has variables for the database host, username, and password. Then, you can use Ansible to fill in the values of the variables and create the configuration file on the server.

How do they work:

* First by reading in the source template file (eg. .j2 file)
* Then by injecting the values of the variables into the template
* Finally deploying the processed file with variables values in place on the remote server. 

Eg: 
```yaml
 - name: Ensure apache is installed and started
  hosts: web
  vars:
    http_port: 80
    http_docroot: /var/www/mysite.com

  tasks:
    - name: verify correct configuration file is present
      ansible.template.template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
        
```
Live Example: [`templates/motd.j2`](templates/motd.j2) and  [`setup-playbook-V5.yml`](setup-playbook-V5.yml)


## Ansible Roles: 

* A method to modularize the different components of the your automation. Think of them as pre-packaged instruction  that group tasks, variables, and handlers together into reusable components.
* Roles are the most efficient way to organize and reuse Ansible automation.
* They allow you to break down complex automation into smaller, more manageable pieces.

Here how they break down the components:

* Task: the core of your automation, defined step-by-step in tasks/main.yml
* Variables: Configrations you can adjust, store in vars/ or defaults.
* Files: static files/resouces like scripts or config files you need to deploy, stored in the files/ directory.
* Templates: Dynamic files that can be customized using variables, located in the templates/ directory.
* Handlers: Actions that run only when notified, defined in handlers/main.yml. 

This Structures approach makes it easier to organize, reuse, and maintain your playbooks and share your automation with others.

With these Ansible Roles we can acheive
* Reusability: write once and use in multiple playbooks 
* Maintainability: components are isolated and self-contained, easier to debug and troubleshoot 
* Scalability: easier to scale your automation
* Collaboration: easier to collaborate with others 

```
my_role/
├── defaults/
│   └── main.yml
├── files/
│   └── readme.txt
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── config.j2
└── vars/
    └── main.yml
```

**How to create and run Ansible roles:**

* Generating a Role
```bash
ansible-galaxy init my_role 
```

* **Tree Example:**

```bash
tree my_role
```

**Tree Structure:**
```
my_role/
├── defaults/
│   └── main.yml
├── files/
│   └── readme.txt
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── config.j2
└── vars/
    └── main.yml
```

# Ansible vs Kubernetes

**Initial Intuition:** *"I think there are some similarities between Kubernetes and Ansible. In Ansible, you can configure web servers, update them, and maintain them. Similarly, Kubernetes has the extra power to spin up pods, install required applications, and manage servers."*

Your intuition is partly correct. There is overlap, but Ansible and Kubernetes solve different problems.

The easiest way to think about it:

**Ansible manages machines. Kubernetes manages applications/containers running on machines.**

### A simple comparison

| Feature | Ansible | Kubernetes |
| :--- | :--- | :--- |
| **Main job** | Configure and automate infrastructure | Run and manage containerized applications |
| **Manages** | Servers, packages, files, services, users, configs | Pods, containers, services, deployments |
| **Typical question** | "How do I configure these 20 servers?" | "How do I keep 10 copies of my app running?" |
| **Works with** | VMs, physical servers, cloud instances, network devices, etc. | Primarily container workloads |
| **Scaling** | You write automation to scale | Built-in concepts for scaling |
| **Self-healing** | You can automate it | Core feature |
| **Desired state** | Yes, to an extent | Fundamental design principle |
| **Example** | Install Nginx and configure it | Run 3 replicas of an Nginx container |

### Where the comparison is right

Suppose you have 10 Linux servers.

With **Ansible**, you might say:

For all 10 servers:
1. Install Docker
2. Install nginx
3. Copy nginx.conf
4. Start nginx
5. Make sure nginx starts on boot

Ansible connects to those machines and performs those tasks. You can also use Ansible to install Kubernetes itself, configure nodes, deploy applications, etc.

With **Kubernetes**, you generally don't say:

* SSH into server 1
* Install nginx
* Start nginx

Instead, you say something like:

```yaml
replicas: 3
image: nginx:latest
```

You're telling Kubernetes:

*"I want 3 instances of this application running."*

Kubernetes then figures out where to run them and continuously works to make reality match that desired state.

If one pod dies:

**You:** I want 3 nginx pods

**Kubernetes:**
* pod 1 ✓
* pod 2 ✓
* pod 3 ✗
* → create replacement pod
* pod 1 ✓
* pod 2 ✓
* pod 4 ✓

That's one of the big differences.

## Ansible Collections: 

* Ansible Collections are a way to distribute and manage Ansible content (modules, plugins, playbooks and roles) in a modular and organized manner.
* Groups components like roles, modules, plugins, etc into a single unit 
* Distribute via Ansible Galaxy or Automation Hub
  * Ansible Galaxy: is a community platform to discover, share and downloada automation content like roles, modules, etc. 
    ```bash
    ansible-galaxy collection list #List all the collections installed on the local system. 
    ansible-galaxy collection install community.general # Install community.general collection.
    ansible-galaxy collection install community.mysql # Install community.mysql collection.
    ```
  * Ansible Automation Hub: is a private registry to store and share automation content within an organization. hosted source of Red Hat, Validated and Certified Partner Contents collections

* Improves Content management and collaboration within teams  


**File Structure:**

```
<namespace>-<collection>/                 # Collection root
├── ansible_module.py               # A module
├── ansible_playbook.yml          # A playbook
├── ansible_role/                   # A role
│   ├── defaults/main.yml
│   ├── handlers/main.yml
│   ├── tasks/main.yml
│   ├── templates/config.j2
│   └── ...
├── ansible_vars/main.yml             # Collection-wide variables
├── meta/collection-info.yaml       # Metadata about the collection
└── README.md                       # Documentation
```

# Creating ansible collections

```bash
ansible-galaxy collection init my_namespace.my_collection --init-path ./ansible_collections/ # this will create the collection in the current directory ( ./ansible_collections/ ) 
```
What is namespace:
* When we have same collection names from different vendors we can distinguish them using namespace.
* In above `my_namespace` is namespace and `my_collection` is collection name. 
* Namespace is the name of the organization or individual who created the collection.

Tree looks like this 
```
./ansible_collections/my_namespace/my_collection/
├── docs
├── galaxy.yml
├── meta
│   ├── runtime.yml
├── plugin
│   ├── README.md
├── README.md
├── roles

``` 
