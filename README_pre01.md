# Obtaining and installing the lab tools on non-Windows 7 platform

You have done these installation steps, you may go to the next section. You only do these tasks once.

If you have a Microsoft Windows 7 platform, do not perform these steps. IBM can set up a vm-based environment for your use in the labs.

To install Container/Kubernetes related tools on your local machine, 

1. Docker Toolbox (select your platform from the left-side menu for instructions for Linux, MacOS, Microsoft Windows): https://docs.docker.com/install/
2. Git scm ( if installing on Windows, the git bash shell can be handy too ) https://git-scm.com/downloads
3. IBM Cloud stand-alone CLI: https://cloud.ibm.com/docs/cli/reference/ibmcloud?topic=cloud-cli-install-ibmcloud-cli
4. Installing plugins for the CLI. After installing the IBM Cloud CLI, run these two commands to install the kubernetes and container registry plugins:
    ```
    ibmcloud plugin install kubernetes-service -r Bluemix    
    
    ibmcloud plugin install container-registry -r Bluemix
    ```

    For both of these binaries, copy the code (after unpacking if it is a tarball) to your system path.

5. Install kubectl (our clusters will be set with 1.13.6 - but latest of 1.14 will be ok) https://kubernetes.io/docs/tasks/tools/install-kubectl/
6. Install helm https://github.com/helm/helm/releases/tag/v2.14.0



