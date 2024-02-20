# How contribute to the website
This website is meant to be filled by the members of CCS Amsterdam. It was set up so that it is easy for everyone to contribute information about their latest news, themselves, their projects, publications, or resources they want to share with the world. This repository is public but you do need to be member of ccs-amsterdam to actually make changes to the website and every new deployment will be checked by the admins. In the following you can find a quick guide on how to make changes to the website:

## Getting started
1) Clone the GitHub repo
2) In the repository, run `npm install` (if this does not work you might need to [install Node.js](https://nodejs.org/en/download))
3) Run `npm start run`
4) If you now go to http://localhost:1313 in your browser, you should see the current version of the website

In general, most of the things that can/should be modified are written in Markdown. This whole website is based on the [Hugo framework](https://gohugo.io). Feel free to add additional themes that have custom shortcuts for things you want to do, their documentation is quite good. Want to add a dark mode? Want to add pictures/backgrounds or make things prettier? Want to add search functions or automated updates? Well, if you are reading this you are likely a computational methods scholar, so feel free to figure it out yourself and go wild :) For everyone who just wants to contribute "normal" things to the page, instructions are below. 

## Adding News
1) Go to [content/en/news_blog](content/en/news_blog)
2) Here you can find an example of a news item (the one not titled _index.md). Duplicate it and insert your own content
3) Added bonus: In [content/en/data](content/en/data) you can find an authors.yaml file. Please insert your info and use your author name for the news entry so everyone can see who wrote it.

## Modifying information about a person/adding a person
Not happy with the picture or content shown about yourself or you want to add a new team member? No problem. 
1) Go to [content/en/persons.md](content/en/persons.md)
2) Search for your own entry and modify it (please, no additional titles or other fancy stuff..) OR add a new person (please maintain first name alphabetical order...)
3) If you want to change your image, add the image to [static/img/people](static/img/people) and either replace the old one or make an additional one and change the name in the `img src` shortcode that is part of your name.

## Adding a current/completed project
1) Go to [content/en/completed_projects](content/en/completed_projects) or [content/en/current_projects](content/en/current_projects)
2) Here you can find examples of project pages (the ones not titled _index.md). Duplicate one of them and insert your own content
3) If you also want this project to appear in the dropdown menu, go to [config/_default/languages.yaml](config/_default/languages.yaml) and add a new item. The structure should become clear from the file: You need to have an identifier (usually project-1 or sth), a name (title), a url which matches what you called the file (e.g. /current_projects/a_promise_is_a_promise), a weight (how high up the item should show up in the menu), a post (short description) and a parent (projects or projects-archive).

## Modifying/Changing/Adding to the publications
1) The publications are from our public [Zotero group](https://www.zotero.org/groups/computationalcommunicationamsterdam) (if you are not part of it now, you should fix that!). Modify the Zotero entries or add new ones.
2) Make sure you have the [Better BibTex extension](https://retorque.re/zotero-better-bibtex/) installed in Zotero (total life saver!). Once you are done modifying the library, export it as "Better CSL JSON" file, name it "bib.json" and replace the current one in the [content folder](content/bib.json). Everything should update automatically.

## Adding things to the resources
1) Go to [content/en/resources](content/en/resources)
2) Here you can find an example of a resources item (the one not titled _index.md). Duplicate it and insert your own content
3) There is one text example but also one with a Youtube video embedded. Be as creative as you'd like to be. 
  
