# How to do cloud A2:

Start by opening ubuntu/wsl (I used wsl)

Enter the command provided here to install teleport:
https://goteleport.com/download/  
The command should look something like:  
```
curl https://goteleport.com/static/install.sh | bash -s 14.1.5
```

Next navigate to you user in cli and create folder(Not sure if this part is 100% zaroori, but I did it so like just do it):  
For me it was:   
```
cd /mnt/c/Users/Usman  
mkdir cloudA  
cd ./cloudA  
```

Next in the directory create 4 files, these files tell teleport how to run and the rules etc it needs to follow, essentially just intructions for the tctl:  
- teleport.yaml (keep this name the same)
- developer.yaml
- auditor.yaml
- aleem.yaml *(use this file to create the dev and auditor files, just change the permissions around, for auditor only allow read and list, and similar for dev, make sure you change the roll names tho lol)*

### Teleport.yaml:

```
teleport:
  data_dir: /var/lib/teleport

auth_service:
  enabled: true
  cluster_name: "teleport-cluster"
  listen_addr: 0.0.0.0:3025
  tokens:
    - "proxy,node,app:accfa21cb720f6015b553a8d3b45f883"

proxy_service:
  enabled: true
  listen_addr: 0.0.0.0:3023
  web_listen_addr: 0.0.0.0:3080
  tunnel_listen_addr: 0.0.0.0:3024
  https_keypairs:
    - key_file: /mnt/c/Users/Usman/cloudA/teleport.key #add your directroy
      cert_file: /mnt/c/Users/Usman/cloudA/teleport.crt #add your directroy

ssh_service:
  enabled: true
  labels:
    environment: "development"
```

### aleem.yaml:

```
kind: role
version: v3
metadata:
  name: aleem_role
spec:
  options:
    max_session_ttl: 8h0m0s
    forward_agent: true  # Enable if the user needs to forward SSH keys
  allow:
    logins: ['walle']  #write your system users name here
    node_labels:
      '*': '*'  # Adjust as necessary to restrict/allow access to specific nodes
    rules:
      - resources: ['session', 'event']
        verbs: ['list', 'read']
      - resources: ['node']
        verbs: ['list', 'read', 'update', 'create']  # Permissions for file management
  deny:
```

Ok now this is done, open 2 terminals, in one terminal write the following command:
```
sudo teleport start
```
This will have started your teleport server, move the the next terminal and leave this be.

Add tokens:

```
sudo tctl tokens add --type=node --ttl=8h
```

Create your admin user(replace bokhari with your name or whatever):
```
sudo tctl users add bokhari --roles=editor,access
```

Now create roles using the files we defined:
```
sudo tctl create -f ./developer.yaml
```
*Repeat this step for all 3 role files*

Create users, when creating users for each role you willg get a link in the responce of the command, open it before creating another user, set the account up, I use google authenticator, but you can use any, save the password etc, for the m.aleem@nu.edu.pk account you make the password should be your full name:
```
sudo tctl users add m.aleem@nu.edu.pl --roles=aleem_role
```


Install Ngrok:
```
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null && echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list && sudo apt update && sudo apt install ngrok
```

Then make sure you have created and Ngrok account online, go to their website and make one if not, once done sign in and in the left navigtion bar thing, there is an option that says "Your Authentication", open it and copy the first command, the command looks like:
```
ngrok config add-authtoken 2YLwM4E9v4BBjObET7Da4jamOGS_3PzMYAm1EzVoUAueJ6Foh
```
Once you've done this run the command:
```
ngrok http https://localhost:3080 --host-header=rewrite
```

Once you do this you'll get a link you can access from another device too, open the aleem account click connect, and then you can use your terminal from an external host. Thats it, ab report bano aur so jao.
