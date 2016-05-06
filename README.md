# BounceBack

[BounceBack][heroku]

[heroku]: http://bounce_back.work

BounceBack is a full-stack messaging app that allows many users to chat together in real time. The goal is to create community where job searchers can interact with companies, look for advice, and socialize. The backend is built using Ruby on Rails with a PostgreSQL database.   The frontend is built on Facebook's ReactJS using Flux architecture and the design is done using Google's Material Design Lite.

fuzzy search
guest login
Component nesting

## Features & Implementation

### Guest login

BounceBack implements anonymous guest users as first class users. This both fits with its mission, allowing users to discreetly discuss details of their jobs or offers, and solves several technical problems. Since guest users are no different than any other user multiple guests can be logged in at the same time. This creates a smoother experience for those interested in trying BounceBack without signing up.

Guests are created through a custom rails route that calls a custom creation method in the `User` model. Upon creation, guests are assigned a unique 4 digit identifier. This differentiates messages from different users and allows a reasonable number of guest accounts to be created. In order to prevent guest accounts from being reused their password_digest is set to "guest". Since BounceBack uses `BCrypt` to secure passwords, setting the password_digest manually precludes the existence of a valid password. Once a guest account is logged out, no one can access that account again.

```ruby
#This method generates new guest accounts in the User model
def self.guest_user
  username = nil

  while(username.nil? || User.find_by(username: username))
    username = "Guest" + rand(10000).to_s.rjust(4, "0")
  end

  guest_user = User.create!(username: username,
                            fname: "guest",
                            lname: "guest",
                            email: username,
                            user_type: "guest",
                            password_digest: "guest")
end
```

### Single-Page App

FresherNote is truly a single-page; all content is delivered on one static page.  The root page listens to a `SessionStore` and renders content based on a call to `SessionStore.currentUser()`.  Sensitive information is kept out of the frontend of the app by making an API call to `SessionsController#get_user`.

![image of messaging layout](https://github.com/a-paulson/BounceBack/tree/master/docs/app_view.jpg)


### Note Rendering and Editing

  On the database side, the notes are stored in one table in the database, which contains columns for `id`, `user_id`, `content`, and `updated_at`.  Upon login, an API call is made to the database which joins the user table and the note table on `user_id` and filters by the current user's `id`.  These notes are held in the `NoteStore` until the user's session is destroyed.  

  Notes are rendered in two different components: the `CondensedNote` components, which show the title and first few words of the note content, and the `ExpandedNote` components, which are editable and show all note text.  The `NoteIndex` renders all of the `CondensedNote`s as subcomponents, as well as one `ExpandedNote` component, which renders based on `NoteStore.selectedNote()`. The UI of the `NoteIndex` is taken directly from Evernote for a professional, clean look:  

![image of notebook index](https://github.com/appacademy/sample-project-proposal/blob/master/docs/noteIndex.png)

Note editing is implemented using the Quill.js library, allowing for a Word-processor-like user experience.

### Notebooks

Implementing Notebooks started with a notebook table in the database.  The `Notebook` table contains two columns: `title` and `id`.  Additionally, a `notebook_id` column was added to the `Note` table.  

The React component structure for notebooks mirrored that of notes: the `NotebookIndex` component renders a list of `CondensedNotebook`s as subcomponents, along with one `ExpandedNotebook`, kept track of by `NotebookStore.selectedNotebook()`.  

`NotebookIndex` render method:

```javascript
render: function () {
  return ({this.state.notebooks.map(function (notebook) {
    return <CondensedNotebook notebook={notebook} />
  }
  <ExpandedNotebook notebook={this.state.selectedNotebook} />)
}
```

### Tags

As with notebooks, tags are stored in the database through a `tag` table and a join table.  The `tag` table contains the columns `id` and `tag_name`.  The `tagged_notes` table is the associated join table, which contains three columns: `id`, `tag_id`, and `note_id`.  

Tags are maintained on the frontend in the `TagStore`.  Because creating, editing, and destroying notes can potentially affect `Tag` objects, the `NoteIndex` and the `NotebookIndex` both listen to the `TagStore`.  It was not necessary to create a `Tag` component, as tags are simply rendered as part of the individual `Note` components.  

![tag screenshot](https://github.com/appacademy/sample-project-proposal/blob/master/docs/tagScreenshot.png)

## Future Directions for the Project

In addition to the features already implemented, I plan to continue work on this project.  The next steps for FresherNote are outlined below.

### Search

Searching notes is a standard feature of Evernote.  I plan to utilize the Fuse.js library to create a fuzzy search of notes and notebooks.  This search will look go through tags, note titles, notebook titles, and note content.  

### Direct Messaging

Although this is less essential functionality, I also plan to implement messaging between FresherNote users.  To do this, I will use WebRTC so that notifications of messages happens seamlessly.  
