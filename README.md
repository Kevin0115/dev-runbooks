
# dev-runbooks
I've spent a lot of hours learning how to deploy stuff. Namely, React applications to GitHub Pages (and then to a custom domain or subdomain), and backend Node servers to free online hosting services. For my reference (and maybe for others who need it), I'll be establishing some standard runbooks for these processes.

## Deploying a React App to GitHub Pages
1. Do the usual `create-react-app` stuff inside of a git repository.
2. Install Github Pages by running `npm install --save-dev gh-pages` inside of the project directory.
3. Add the following lines to the `scripts` section of `package.json`<br />
_If it is a personal/user page meant to be hosted at `user.github.io`:_
`"predeploy": "npm run build",`
` "deploy": "gh-pages -b master -d build",`<br />
_If it is a project page meant to be hosted at `user.github.io/yourproject`:_
`"predeploy": "npm run build",`
`"deploy": "gh-pages -d build"`<br />
_Note the difference is in which branch `gh-pages` publish the static build code to. For a user page, this must be the `master` branch, and for a project page this will be the `gh-pages` branch._
**Sidenote**: _Ever since the renaming of the `master` branch to `main` branch, this restriction on user page build branch may not apply anymore. I haven't tested it out yet._
4. Add the following line at the top level of the `package.json` file in the project directory:
If it's a **project page**: `"homepage": "https://user.github.io/yourproject",`
If it's a **user page**: `"homepage": "https://user.github.io",`
5. Run `npm run deploy` in the project directory; internally, this will create a product build of your React app, and publish it to GitHub in a different branch than your source code.
6. At this point, your user page can be found at `user.github.io` (or a project page at `user.github.io/yourproject`. You can take this one step further by using some DNS magic to utilize a custom domain of your choice.

## Deploying a GitHub Page to a Custom Domain
There are guides for this already. I'll just list the important stuff, as well as some time saving shortcuts.
*Prerequisite*: Ownership of a custom domain. If you don't own one, you can claim ownership of one for pretty cheap from the registrar of your choice. I like [Google Domains](https://domains.google.com/registrar/), but any reliable registrar should let you fiddle with a domain's DNS settings.<br/>
**Important Consideration**
This mostly applies to publishing your user page (`user.github.io`) to a custom domain, so that it is accessible through `yoursite.com` or `www.yoursite.com`. By *default*, project pages (like `user.github.io/yourproject`) will just redirect to `yoursite.com/yourproject`. However, if you would prefer to host your project page on a subdomain of your root domain (such as `yourproject.yoursite.com`, there will be special instructions below the main instructions. You should probably do your own research as to whether you should use subdomains or subdirectories for project pages (things to consider like SEO) - in my opinion though, it's really just about personal flavour.<br/>
1. Add a `CNAME` file to the `/public` directory in the source code branch (as opposed to the `gh-pages` build branch) of your React project (this is at the top level). This file has no extension like `.txt` or anything. It should contain your desired domain (with desired subdomain). Some examples include:
`www.yoursite.com`
`blog.yoursite.com`
`project.yoursite.com`
*It's worth noting* that you should **NOT** include a CNAME for a project page that you wish to exist as a subdirectory page like `yoursite.com/project`. Only include it if you want it to exist on its own subdomain like `project.yoursite.com`, and follow the special instructions in the next section to set up the subdomain stuff.
2. Now, when you run `npm run deploy`, this `CNAME` file is included in every build. Go to the Settings tab of your repository, and make sure the "Custom Domain" field has been automatically filled in with the same contents as your `CNAME` file; if not, just fill it in yourself.
3. While you're at the Settings tab of your repository, it is worth ensuring that GitHub Pages is building off the correct branch (`master`/`main` for user page, `gh-pages` for project page). This should probably have been done in the previous section if there was any troubleshooting involved with getting a project page deployed.
4. Moreover, it's worth checking the "Ensure HTTPS" box in the GitHub Pages section of the Settings tab. If this shows a warning/error, wait until after you configure your DNS settings before addressing it.
5. Now, log into the registrar where your owned custom domain is hosted. In the DNS settings, you're going to add a few "Custom Resource Records".
6. Add an `A` record with the following details:
Name: `@`
Type: `A`
Data/Address (you need all four): `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
7. Add a CNAME record with the following details:
Name: `www`
Type: `CNAME`
Data/Address: `user.github.io.` (Obviously, replace *user* with your own GitHub username. Also, that **trailing dot** is absolutely necessary)
8. Once these are added, give it a moment (could be minutes, could be hours) for the DNS records to update in the magical cloud of DNS wizardry. You should be able to visit `yoursite.com` or `yoursite.com/yourproject` directly to view your GitHub pages.<br/>
### Special Instructions for Deploying GitHub Project Page to a Custom Subdomain
Like I mentioned before, you have the option to host your project page on a subdomain or your custom domain, like `project.yoursite.com` instead of `yoursite.com/project` (which is the default behaviour if you followed the above steps). Personally, I think it's nice to have subdomains for different projects since they appear to exist as separate applications, outside of each others' contexts. But it's ultimately up to you; if you prefer subdirectories, just ignore this section.
1. One thing you'll want to remove is the `homepage` field of your `package.json` of your project directory. I'm not 100% sure how this works, but it was something I discovered while debugging this subdomain deployment.
2. As mentioned in the section above, it's necessary to include a `CNAME` file with the desired subdomain name (such as `project.yoursite.com`) in the `/public` directory of your project.
3. Go to your registrar again and add one more custom DNS record for your domain, with the following information:
Name: `project` (replaced with whatever project name/subdomain you want)
Type: `CNAME`
Data/Address: `user.github.io.` (Replace *user*; trailing dot *still* necessary)
4. Run `npm run deploy` in the project root directory, let the build finish, and visit your defined subdomain (it might take a while if you just configured your DNS settings.
5. Eventually, you should see your GitHub project page deployed on your custom subdomain<br/>
*Additional Note*: If you try to access your project page via the subdirectory URL (`yoursite.com/project`), it will actually just redirect you to `project.yoursite.com`; this was a nice discovery as it didn't break all the existing links to my project pages.
