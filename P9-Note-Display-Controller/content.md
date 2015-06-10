---
title: "Note Display Controller"
slug: note-display-controller
---     

Time to move onto displaying a note in its own custom controller. We will be creating a reusable view controller that will be used to display our note information
but can also be used to manipulate notes using keyboard input.

##Adding a Container View

Let's add a new container view to our `New Note View Controller`. This container will create a new view controller that will be displayed within the `New Note View Controller`.

> [action]
> Open `Main.storyboard` and locate `Container View` in the *Objects Library*
> Drag this into the `View` of your `New Note View Controller`, then resize it vertically to sit under the navigation item bar.
> Notice that there's now a new view controller in your storyboard, connected to the `Container View`.
> Rename this new controller to `Note Display View Controller` by selecting it in Document Outline and pressing Enter.

When the container was added, it created a new embed segue under `New Note View Controller` named "Embed segue to View Controller."

> [action] 
> Set this segue identifier to: 'ShowNewNote' (We will be using this later on).
> ![image](embed_seque_1.png) ![image](embed_seque_2.png) 

##Adding the Note Display Controller

> [action] 
> As per the previous chapter, create a new View Controller subclass entitled `NoteDisplayViewController` and set your newly added View Controller to use this Custom Class.

Let's add a bit of initial usability to our new Note Display View Controller.

> [action]
> 1. Go back into `Main.storyboard`, locate `Toolbar` in the *Objects Library* and drag this into your empty `Note Display View Controller`. 
> 2. Add the following *Pin* constraints as we want to ensure our `Toolbar` sits at the bottom of the view.
> ![image](toolbar_constraints.png) 
>
> 3. Select the `Item` object in our `Toolbar` and change the `Identifier` to Trash. You will see `Item` change into a trash can icon.
> 4. Connect the Trash icon to the `Exit` of the View Controller. You will be presented with a popup to select the `unwindToSeque` action.
>
> ![image](connect_trash_exit.png) 
>
> A new seque will have been created.
> Set up the Identifier as 'Delete', you will be using that in the switch statement in the `unwindToSeque` function.
>
> ![image](display_seque_exit_1.png) ![image](display_seque_exit_2.png) 

We also want a way to call this `Note Display View Controller` when a row is selected so we can display our note.

> [action]
> Connect (Ctrl-Drag) your Dashboard View Controller to the `Note Display View Controller`.

![image](manual_seque.png) 

This will create a new segue called Show segue to Note Display View Controller.

> [action]
> Set the identifier to 'ShowExistingNote'.
 
![image](manual_seque_select.png) ![image](manual_seque_attributes.png) 

Great, let's add support for our new trash can segue.

> [action]
> Open `NotesViewController.swift` and add the following to the `switch` statement in your `unwindToSeque` function.
>	
	case "Delete":
	    realm.transactionWithBlock() {
	        realm.deleteObject(self.selectedNote)
	    }
>        
        let source = segue.sourceViewController as! NoteDisplayViewController
        source.note = nil;
>
The trash can, when clicked, will now delete notes.

Time to enable the table row selection to trigger the segue to the `Note Display View Controller`.

> [action]
> Uncomment the following code in your `UITableViewDelegate` extension in NotesViewController.swift.
>
	self.performSegueWithIdentifier("ShowExistingNote", sender: self)
	
Ah, those good handy segue identifiers...

##Bonus
You may have noticed we are now performing a `Delete` operation in two seperate places. This seems like a good candidate to refactor and ensure we have a unified function that takes 
a note and deletes it. This can then replace both chunks of `Delete` code.

> [solution]
> 
   func deleteNote(note: RLMObject?) {
        let realm = RLMRealm.defaultRealm()
        realm.transactionWithBlock() {
            realm.deleteObject(self.selectedNote)
        }
    }
>
 

##Displaying the Note

Time to create an interface to present our Note information and move us towards being able to edit this information. 
> [action]
> 1. Open `Main.storyboard` and locate your `Note Display View Controller Scene`
> 2. Add a `Scroll View` to your main `View`. Your notes have the potential to contain a lot of content, so you want to ensure the user can scroll through them. Be sure to resize it to fit the space between the navigation
item at the top and the toolbar at the bottom.
> 3. Add a `Text Field` to your `Scroll View` near the top. This will be used to display the title.
> 4. Add a `Text View` in your `Scroll View` and add it below your `Text Field`. This will be used to display the content.

We have a basic presentation interface, now we need to connect the `Text Field` and `Text View` objects with our `Note Display View Controller`. 

It should look like this:

![image](display_view.png) 

Remeber if things are not looking quite right when you run on device, you can generally solve these through resolving auto layout that you tried out
in the simple app tutorial.

>[action]
> Select from the main menu `Editor\Resolve Auto Layout Issues\(Selected Views) Reset to Suggested Constraints`>
>
> ![image](reset_constraints.png)
 
Time to add some outlets.

> [action]
> Open `NoteDisplayViewController.swift` and add modify the head of your class to read as follows:
>
	import Foundation
	import UIKit
	import Realm
	import ConvenienceKit
>
	class NoteDisplayViewController: UIViewController {
>	    
	    @IBOutlet weak var titleTextField: UITextField!
	    @IBOutlet weak var contentTextView: TextView!
>        
	    override func viewDidLoad() {
>

For the eagle eyed, you will notice that we are using `TextView` and not `UITextView` this is provided by the `ConvenienceKit` framework, 
this framework was created by Make School as an input helper to streamline the process of handling user input. *Cmd-Click* if you are curious about this subclass.

> [action]
> 1. Update your `UITextView` object in the *Identity Inspector* to use the `TextView` class.
> 2. Set `Module` to `ConvenienceKit`
> 3. Connect your `UITextField` and `TextView` Interface Builder objects to the `IBOutlets` variables in your code.  
> In case you have forgotten, here is a little reminder:
>
> ![image](code_connect_fields.png) 
>

Now let's get these new display objects display our existing note information, we need to add a variable to hold our Note. 

> [action]
> Add the following note declaration after the `contentTextView` declaration.
>
	var note: Note? {
        didSet {
            displayNote(self.note)
        }
    }
>
	
Remeber the `didSet` functionality we added during the 'Local Storage with Realm' chapter? Have a quick look at `NoteTableViewCell` for a reminder.
In this case when a note is set, we want to call the `displayNote` method to populate our title and content display objects in our view.

> [action]
> Add the following function to the end of your class before the closing squiggley.
>
	func displayNote(note: Note?) {
        if let note = note, titleTextField = titleTextField, contentTextView = contentTextView  {
            titleTextField.text = note.title
            contentTextView.text = note.content
        }
    }
>	
> That should do the trick, ensure all variables are not nil and then set the objects with the Note data.  

##Sharing Note Data
How can we send the Note information across to this controller? 

Well, once we can use segue functionality, when a segue is being prepared you can override the functionality and perform your own actions.
In this case we will override the `prepareForSegue` functionality, check for the `ShowExistingNote` identifier and then set our `Note` variable in our `Note Display View Controller` 
with the currently selected `Note` in our `Table View`.  Sound easy? :)

> [action]
> Open `NotesViewController.swift` and add the following code at the end of our class (before the extensions).
>
	override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        if (segue.identifier == "ShowExistingNote") {
            let noteViewController = segue.destinationViewController as! NoteDisplayViewController
            noteViewController.note = selectedNote
        }
    }
>
	
This does exactly what is described.  It checks for the given identifier and then grabs our reference to the `destinationViewController`, which we expect to be a `NoteDisplayViewController`.
This gives us access to the `note` variable in this controller and finally we can set it to the currently selected note.

Now what will happen is the note will get set and `didSet` will be called. However, the display objects have not yet been created and hence are nil, so no information can be presented.

Now we go back to our `Note Display View Controller` and ensure we call `displayNote` once the view is ready for action.

> [action]
> Ensure your `Note Display View Controller` code is as follows:
>
	import Foundation
	import UIKit
	import Realm
	import ConvenienceKit
>
	class NoteDisplayViewController: UIViewController {
>    
	    @IBOutlet weak var titleTextField: UITextField!
	    @IBOutlet weak var contentTextView: TextView!
>	    
	    var note: Note? {
	        didSet {
	            displayNote(self.note)
	        }
	    }
>	    
	    override func viewWillAppear(animated: Bool) {
	        super.viewWillAppear(animated)
>
	        displayNote(self.note)
	    }
>	    
	    //MARK: Business Logic
>	    
	    func displayNote(note: Note?) {
	        if let note = note, titleTextField = titleTextField, contentTextView = contentTextView  {
	            titleTextField.text = note.title
	            contentTextView.text = note.content
	        }
	    }
>   
	}
>

##To Load or Appear?

Notice we are calling `displayNote` in `viewWillAppear` rather than the previously supplied `viewDidLoad`.  What is the difference you may ask yourself?

 - **viewDidLoad** is called once upon initialization. 
 - **viewWillAppear** is called every time the view is about to be displayed. This ensures it is always refreshed.

We are going to make one more modification before running our app and trying out all these changes.

When we select the `New Note View Controller` we want to be able to create an empty `Note` and hold onto it for saving later, but also make it accessible in the `Note Display View Controller`.

> [action]
> Open `NewNoteViewController` and modify the `prepareForSegue` function to read as follows:
>
	override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        // Get the new view controller using segue.destinationViewController.
        // Pass the selected object to the new view controller.
>        
        if (segue.identifier == "ShowNewNote") {
            // create a new Note and hold onto it, to be able to save it later
            currentNote = Note()
            let noteViewController = segue.destinationViewController as! NoteDisplayViewController
            noteViewController.note = currentNote
        }
    }
>
	
This should be very familiar from the previous application of this logic in `NotesViewController.swift` only a few chapters ago.

Time to run the app! 

Hopefully you have a few 'Super Simple New Notes' left over. Click one! 
\o/ your title and content should be displayed!
Don't worry if it's a bit ugly, functionality first before aesthetic beauty.

Try and create a new Note. You can edit the title and content however it will not save it just yet. Hit *Save* and you will have created a new note. However,
it has no title or content, only its modification date. Boooo.

![image](simulator_dashboard.png) ![image](simulator_view.png) ![image](simulator_add.png) 

##Enabling Keyboard Input

In the simulator by default it will not show the iOS keyboard. You can simply type into the fields with your physical keyboard, which tends to make input easier when testing.  
However, I find it easier to disable the physical keyboard so it will always default to the software keyboard to get more accurate simulation.
From the `iOS Simulator` menu: `Hardware/Keyboard/Connect Hardware Keyboard` to deselect this option.

Let's add quick support for modification of our notes.

When is a good time to save a Note? When the View is dismissed seems like a good place to do so.

> [action]
> Add the following code to `NoteDisplayViewController` after `func displayNote`:
>
    override func viewWillDisappear(animated: Bool) {
        super.viewWillDisappear(animated)
>        
        saveNote()
    }
>   
    func saveNote() {
        if let note = self.note {
            let realm = RLMRealm.defaultRealm()
>            
            realm.transactionWithBlock {
                if (note.title != self.titleTextField.text || note.content != self.contentTextView.textValue) {
                    note.title = self.titleTextField.text
                    note.content = self.contentTextView.textValue
                    note.modificationDate = NSDate()
                }
            }
        }
    }
>
    
When you press the back button, e.g. '< Dashboard', the view will be dismissed and the `viewWillDisappear` method will be called.  
At this point `saveNote` is called. This method will check that the fields have changed. If so, it will update the note.

If we return back to our `Dashboard Scene` at this point, there will be no update.

As we learned in this chapter, you should refresh scene information in `viewWillAppear`.  Time to go back and make this change to our `Dashboard Scene`.

> [action]
> Modify `NotesViewController` as follows:
>
    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.dataSource = self
        tableView.delegate = self
    }
>    
    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
>
        notes = Note.allObjects().sortedResultsUsingProperty("modificationDate", ascending: false)
    }
>    

Now run the app. You can add new notes and edit existing notes using the physical keyboard.  Another step closer to full Note management!

Great time to **Commit your code**

Give yourself a pat on the back! The app is coming along nicely. Time to move onto the next chapter and get a bit more involved with Keyboard Handling.
