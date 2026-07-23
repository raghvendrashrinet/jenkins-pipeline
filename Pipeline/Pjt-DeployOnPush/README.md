

1. - Create a Jenkins job named devops-app-deployment and configure it so that if anyone pushes any new change to the origin repository in master branch, the job should auto build and deploy the latest code on App Server 1 under /var/www/html directory.
Before deployment, ensure that the ownership of the /var/www/html directory is set to user sarah, so that Jenkins can successfully deploy files to that directory.


2. - SSH into App Server 1 using sarah user credentials mentioned above. Under sarah user's home (/home/sarah/web) you will find a cloned Git repository named web. Under this repository there is an index.html file, update its content to Welcome to the xFusionCorp Industries, then push the changes to the origin into master branch. This push must trigger your Jenkins job and the latest changes must be deployed on the server,
4. - also make sure it deploys the entire repository content not only index.html file.
