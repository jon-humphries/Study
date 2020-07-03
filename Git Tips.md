# **Git Tips**

### Create Initial Git Repo

**Logon to GitHub and Create a new repo then:**


	echo "# Study" >> README.md
	git init
	git add README.md
	git commit -m "first commit"
	git remote add origin https://github.com/pathtorepo.git
	git push -u origin master
                
**To list the available branches for your current project, type in :**

	git branch 

**To create a new branch, naming it whatever you want, type in :**

	git branch branch_name 

**To delete a branch, type in :**

	git branch -D branch_name 

**To switch to a branch, making it the currently active branch, type in :**

	git checkout branch_name 

**As an example, in order to return to your master branch, you would type in
**
	git checkout master
