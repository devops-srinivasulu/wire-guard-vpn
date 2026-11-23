Understood. You want to **remove the existing Backend agent connection completely and connect it again**.

Follow these steps.

### Step 1: Stop the existing agent
On the Backend EC2 server, run:

```bash
sudo pkill -f 'java -jar agent.jar'
```

Verify:

```bash
ps aux | grep '[a]gent.jar'
```

No agent process should remain.

### Step 2: Delete the existing connection in Jenkins
1. Open `https://jenkins.sankranti.com`.
2. Go to **Manage Jenkins → Nodes**.
3. Select **backend**.
4. Click **Delete Agent**.
5. Confirm deletion.

**Important:** This deletes the Jenkins node configuration. Make sure you have noted its existing settings before deleting:
- Node name: `backend`
- Remote root directory: `/home/jenkins/agent`
- Labels
- Number of executors
- Launch method

### Step 3: Create the agent again
1. Go to **Manage Jenkins → Nodes**.
2. Click **New Node**.
3. Enter `backend` as the name.
4. Select **Permanent Agent**.
5. Configure the settings using the values you noted earlier.
6. Save.

### Step 4: Connect the agent
Open the newly created `backend` node.

Use the **Launch agent** instructions shown by Jenkins to connect it from your Backend EC2 server.

### Step 5: Verify

```bash
ps aux | grep '[a]gent.jar'
```

Then check Jenkins. The backend node should show **Online**.

**One caution:** Don't delete the agent until you've recorded its existing configuration and confirmed you can retrieve its connection secret again.
