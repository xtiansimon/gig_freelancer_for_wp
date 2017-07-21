README gig_freelancer_for_wp

# WordPress_setup_for_freelancer
These are instructions for setting up your personal WordPress blog, so you can hire a gig freelancer to pull out a thorn.

These instructions assume your personal blog is on a low-cost Shared Hosting server. For these instructions you will require privledged access to your remote server:

    - SSH/Bash access to your shared server, noted below as `$`
    - cPanel, noted below as `cPanel`
    - WordPress-Admin, noted below as `WP`

This assumes that if your live WordPress directory on your remote server looked like this:

    + /home/mydomain/public_html/
        + fooproject/
            - dev.fooproject/
            - live.fooproject/

Then your WordPress directory will look something like this when finished:

    + /home/mydomain/public_html/
        + fooproject/
            - dev.fooproject/
            - freelancer.fooproject/
            - git.fooproject/
            - live.fooproject/


## Create central repository
1. Create bare git central repo:
Begin in the project directory on your webserver. This is the parent directory of your WordPress install directory:

    $ cd /home/mydomain/public_html/fooproject/
    $ git init --bare fooproject.git

(NOTE. This is how I like manage directories on my webserver. Your structure will likely be different. The important feature is to locate the central repository in a directory where the freelancer does not have permissions to access via ftp.)

## Create git project repo in existing WordPress install directory
Change directory to the production (or testing/dev) directory:

    $ cd /home/mydomain/public_html/fooproject/live.fooproject/

2. Create `.gitignore` file for the current installation.
There are plenty of examples on the internet. At this point, I'm concerned only with sensitive information 

    # Ignore configuration files that may contain sensitive information.
    wp-config.php
    
3. Create git project repo OR update current git repo
I originally had a git repo with `live` and `dev` versions. After I stopped developing for a time, I went through my subdomains and removed the central repos and `dev` subdomains. For this project, I needed to recreate all of the git repos. 

4. Create new:

    $ git init
    $ git add *
    $ git commit .

OR, Update existing project repo.

5. Updating Remote name and url
Update the project repo to point to the central repo:

    $ git remote add origin path/to/your/central/git/repo

Confirm change:

    $ git remote -v

6. Rename origin to project name
(I don't think this is necessary, but it's something that I've been doing)

    $ git remote rename origin fooproject

7. Push to the Central Repository; Complete initial git repo setup
    $ git push fooproject master

Now we have two copies of the exisiting website. 
Next, we git pull from the central report to our new freelancer directory.


## Clone project to freelancer subdomain 
8. Create new folder in project directory:

    $ mkdir freelancer.fooproject
    
9. Change directory to the `freelancer.fooproject` directory:

    $ cd /home/mydomain/public_html/fooproject/freelancer.fooproject
    $ git clone --branch master fooproject.git freelancer.fooproject

Now there are three copies of the exisiting website. 
This new subdomain we will share with the freelancer.

## Create subdomain in your domain's cPanel

    cPanel [Home] > Domains > Subdomains
        url= freelancer.fooproject.mydomain.com
        points= /home/public_html/fooproject/freelancer.fooproject

## Set up the WordPress database
10. Copy the existing WordPress database:

    cPanel [Home] > Databases > phpMyAdmin
        - Select database in sidebar: `fooproject`
        - In focus area find Operations [tab] --> copy database to
        - Enter new database's name=
            `fooproject_dev`
        
    
11. Edit the `options` table in the new database copy.
Using the subdomain trick, phpMyAdmin will group the new database in the database names tree. 

12. Open the tree to see the new database's tables.

- Select database table:
        `fooproject_dev` > fooproject_options 

13. Edit table values:

    siteurl = freelancer.fooproject.mydomain.com
    home = freelancer.fooproject.mydomain.com
    blogname = [existing title] -- freelancer


## Create new database user.
Warning. Your live WordPress directory has the file `wp-config.php`. This file reveals the database password. Avoid exposing your live passwords. Create new user just for freelancers:

    cPanel [Home] > Databases > MySQL Databases

14. Create new user:

    user = fooproject_freelancer
    password = [same as database]
    priveledges, all

15. Add user as Privileged User of new database:

    - Find on page --> 'Add a User to a Database'
    - You database will only need this one Privileged User


## Customize `wp-config.php` for freelancer subdomain.

16. Copy `wp-config.php` from live/dev site's directory to freelancer WordPress directory.

17. Edit and change the values for the following parameters to match new database user (#4.1):

    DB_NAME
    DB_USER
    DB_PASSWORD


## Create FTP user access.
FTP is helpful when you're developing. Be kind to your freelancer and give them FTP.    

    cPanel [Home] > Files > FTP Accounts
        user=freelancer
        password=[same as database]
        quota = 500MB (tried 5MB, but this was used up just creating the user!)


## Update site permissions.
With all this copying and stuff, it's best to update the projects permissions. It's usually the first cause of 500 server errors.

    $ cd /home/mydomain/public_html/fooproject/freelancer.fooproject
    $ find . -type d -exec chmod 755 {} \;
    $ find . -type f -exec chmod 644 {} \;
    
- Do the same for central repository.


## Add Freelancer User to WordPress.
Your current WordPress user is still the same as on the original live WordPress site. This is very dangerous to your site.

18. Login as admin to edit:

    http://freelancer.fooproject.mydomain.com/wp-admin/

19. Create new WP user, "freelancer".
    
20. Add new user:

    WP [dashboard] > Users > All Users --> Add New
        user = freelancer
        password = [same as database]
    
21. Grant Admin privledges:

    WP [dashboard] > Users > All Users --> Change role to...
        - Check new user name from the list    
        - Select 'Administrator' from `Change role to...` pulldown menu

22. Change your admin user's password also !!
    (I also updated my email address from private to business)


## Disable Server Cache for subdomain: 

    cPanel [Home] > Advanced > Cache Settings
    Disable cache for freelancer.fooproject.mydomain.com

+++
