# Lab 0: Choose your editor

Duration: 15 minutes

You should have received the lab workstation login information from the instructor.

To follow the rest of the labs, you'll need to edit files and run commands on
your workstation. This lab documents three ways to connect to your workstation.
Choose whichever method you're most comfortable with, and follow the steps
below to connect to your workstation.

1. Visual Studio Code

    Visual Studio Code, or VSCode, is a popular open source editor from Microsoft. Using an extension called Remote-SSH, you can connect to your workstation, edit files, and run commands all from within VSCode.

2. SSH

    If you're comfortable using SSH and a terminal-based editor like vim or nano, you can use SSH to connect to your workstation.

3. Web-based SSH Client

    If you don't want to, or can't, install and use VSCode or SSH on your system, you can use a web-based SSH client from within your browser.


## VSCode

One way to navigate through this training is to use VSCode with a few added extensions. There are a few steps to install and configure VSCode, but once set up, this provides an easy to use environment.

- Task 1: Download VSCode
- Task 2: Download the Remote-SSH Extension
- Task 3: Configure SSH
- Task 4: Connection to your workstation in VSCode

### Task 1: Download VSCode to your local machine and configure extensions

1. Follow the instructions from [this site](https://code.visualstudio.com/download) to get the latest official download for your operating system.

1. Get familiar with the VSCode UI. The HashiCorp Configuration Language (HCL) is supported natively with VSCode.

### Task 2: Download the Remote-SSH Extension

1. Install an OpenSSH compatible SSH client if one is not already present.
    - For Windows 10, follow [these instructions from Microsoft](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse).
      Note: PuTTY is not supported on Windows since the ssh command must be in the path.
    - For MacOS, OpenSSH should already be installed. Open a terminal window and run `ssh -V` to make sure.

1. Install the [Remote Development extension pack](https://aka.ms/vscode-remote/download/extension) for VSCode.

### Task 3: Configure SSH

1. Create your local ssh config on your machine.

    Create an ssh config file in `~/.ssh/` or `c:\Program Files\Git\etc\ssh\` called `config` and add your workstation and user information. For example:

    ```text
    Host 18.130.225.52
        User terraform
        HostName 18.130.225.52
    ```

    Don't forget to change the IP address to the IP address of your workstation, and the `User` field to the correct username, both of which will be provided by your instructor.

1. Mac or Linux only: Enable ControlMaster

    VSCode will need to make multiple SSH connections to your workstation. This can mean that it will ask for your password for each connection. To avoid this, you can enable ControlMaster. First, edit your SSH config file:

    ```text
    Host 18.130.225.52
        User terraform
        Hostname 18.130.225.52
        ControlMaster auto
        ControlPath  ~/.ssh/sockets/%r@%h-%p
        ControlPersist  600        
    ```
    Remember to update the IP address and username accordingly.

    Next, create the sockets directory by running this command:

    ```sh
    mkdir -p ~/.ssh/sockets
    ```

Finally, if you're an expert SSH user, feel free to install your public key on the workstation to enable password-free authentication.

### Task 4: Connect to your workstation in VSCode

1. Using the command palette on the left side of the window, choose `Remote-SSH: Connect to Host...`
1. Select your SSH config file
    - For Windows, this is `c:\Program Files\Git\etc\ssh\config`
    - For Mac or Linux, the file is `/home/<your_username>/.ssh/config`
1. Choose the IP of your workstation and enter your password when prompted
1. Open the explorer and choose `Open Folder`
1. Navigate to `/workstation/terraform/`
1. You can also launch a terminal from VSCode for remote command line execution with Ctrl+`

## SSH

If you'd prefer to SSH directly into your workstation, you should already be familiar with how to use SSH and either vim or nano.

On Windows, use an SSH client such as PuTTY or OpenSSH. On a Linux or Mac, use the Terminal and OpenSSH to connect to your workstation.

1. In a terminal window, run:

    ```text
    ssh terraform@<WORKSTATION_IP>
    ```

    Replacing `<workstation_IP_address>` with the IP address and `terraform` with the username provided by your instructor.
1. Enter your workstation password to login.
1. On your workstation, navigate to `/workstation/terraform`.
1. The workstations have both vim and nano installed, so use whichever editor you're most comfortable with.

## Web-based SSH Client

It's also possible that you can't, or would prefer not to, install new software on your machine. In this case, you can use a web-based SSH client that we've installed on your workstation.

1. Launch a web browser and enter:
    ```text
    https://<WORKSTATION_IP>/
    ```
    Be sure to change the username from `terraform` to the one provided by your instructor. You may get a security warning from your browser, because the wetty client uses a self-signed SSL certificate. It's safe to dismiss this warning.
1. When you are prompted, enter the password provided by your instructor.
1. On your workstation, navigate to `/workstation/terraform`.
1. The workstations have both vim and nano installed, so use whichever editor you're most comfortable with.
