
# dev-runbooks
I've spent a lot of hours learning how to deploy stuff. Namely, React applications to GitHub Pages (and then to a custom domain or subdomain), and backend Node servers to free online hosting services. For my reference (and maybe for others who need it), I'll be establishing some standard runbooks for these processes.

## Table of Contents
* [Deploying a React App to GitHub Pages](#deploying-to-github)
* [Deploying a GitHub Page to a Custom Domain](#deploying-to-custom-domain)
	* [Special Instructions for Deploying GitHub Project Page to a Custom Subdomain](#subdomain-instructions)
* [Deploying a Node.js Server to Vercel](#node-to-vercel)
## <a id="deploying-to-github">Deploying a React App to GitHub Pages</a>
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

## <a id="deploying-to-custom-domain">Deploying a GitHub Page to a Custom Domain</a>
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
### <a id="subdomain-instructions">Special Instructions for Deploying GitHub Project Page to a Custom Subdomain</a>
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


## <a id="node-to-vercel">Deploying a Node.js Server to Vercel</a>
If you've built a full-stack JavaScript application, chances are you're running some kind of Node.js server as your backend. Whether you run this locally on `localhost:3000` or on some virtual machine, this tutorial is aimed to introduce you to a free and simple-to-use service that lets you run a variety of server types - though, my primary purpose for using Vercel is just to run my backend servers for my full-stack apps.<br />
*Prerequisites*: From this point, I'll assume that you have a functioning Node.js/Express.js server with RESTful endpoints and all. A suggestion is to have your application listen to port 80 (the port reserved for HTTPS). I'll also assume you have node/npm installed on your machine (how else would you have built a Node server)

1. Go to [Vercel's website](https://vercel.com/) and make a free account. I opted to login with GitHub since it's convenient. During registration it might ask you to link some repositories and grant access to GitHub, but this isn't necessary at all; you don't need any type of GitHub integration to run these servers (*although* Vercel can deploy directly from GitHub whenever a new commit is detected, so if this type of continuous deployment interests you, go for it).
2. Open a terminal window anywhere and run the command `npm install -g vercel`
3. Now that the Vercel CLI is installed, you just need to add a file named `vercel.json` to the root directory of your Node.js server project directory. The standard contents should be:

```
{
	"version": 2,
		"builds": [
			{
				"src": "./index.js",
				"use": "@vercel/node"
			}
		],
		"routes": [
			{
				"src": "/(.*)",
				"dest": "/"
			}
	]
}
```
Of course, you can change these contents to suit your needs, but this is the basic "plug and play" 					configuration that I found works immediately.

4. If you have `nodemon` installed, your `start` script in `package.json` might be changed to something like 
`"start": "nodemon --watch ./*"`. If so, I would change this to the following:
`"start": "node index.js"`, just because it plays better with Vercel.
5. Run `vercel` in a terminal window at the root directory of your project. You'll likely want to set this up as a new project (so assign it an initial project name) and use the defaults (just press enter) for most of the prompts.
6. Vercel will then start a deployment that you can track in the dashboard through their site.
7. Once the deployment is complete, you'll find your server up and running at `yourproject.vercel.app`. If you have a landing site, you should be able to see it just by visiting that URL. Test out some `GET` endpoints by visiting `yourproject.vercel.app/some_endpoint` to make sure it's working correctly.
8. When you make any further code changes, run `vercel --prod` instead of just `vercel` for any future deployments.
