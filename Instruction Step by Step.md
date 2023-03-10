# create the project
1. Create a new project of type Class Library
2. Give it a name you want to use for your Add-In, we will use "OneNoteRibbonAddIn"
3. Register the project for COM interop
   - a. Right Click on the Project and go to Properties
   - b. Click the Build tab on the left
   - c. In the Output section, check the checkbox labeled "Register for COM interop"
   - d. Save and Close

4. Edit the AssemblyInfo.cs file (in the Properties folder)
a. Delete the line that says: "[assembly: ComVisible(false)]"

5. Delete the auto generated "Class1"
6. Create the project structure  
- a. Add the default Resources file  
  - i. Right click the project -> Properties -> Resources -> Click the link to create a default resources file  
  - ii. Resources.resx will get added to your project under the Properties folder.  
  - iii. Save and Close.  
- b. Add a new folder to the project, call it "Resources"
- c. Inside the Resources folder add an xml file, name it "customUI.xml"
  - i. Right click on the file and go to Properties
  - ii. Change the Build Action to None
- d. Add an image, this is the image that will be displayed in the Ribbon, name it "showform.png", Note: file type should be *.png
- e. Add a class, call it "Connect.cs"
- f. Add a class, call it "CWin32WindowWrapper.cs"
- g. Add a class, call it "ReadOnlyIStreamWrapper.cs"
- h. Add a windows form, call it "MainForm.cs"
  - i. Inside the form code, add the attribute [ComVisible(false)] just above the class declaration (public partial class MainForm: Form)
  - ii. Add using System.Runtime.InteropServices;  at the head of "MainForm.cs"
#  Add the installer project
1. Right click the solution -> Add -> New Project…
2. In the project types on the left, expand Other Project Types, then select Visual Studio Installer
3. Name it "OneNoteRibbonAddInSetup"
4. Set the solution to build the installer project
  - Right click the installer project, select Properties
  - In the OneNoteRibbonAddInSetup Property Pages, click the Configuration Manager button in the top right of the form
  - Check the Build checkbox on the setup project line
  - Hit close and ok, to dismiss the forms				
# Add the external references:
1. Right click References, Add Reference…
2. Expand the Assemblies, and select the Extensions type, add "Extensibility", version 7.0.3300.0
3. Select the COM type, add "Microsoft Office 15.0 Object Library", version 2.7, and add "Microsoft OneNote 15.0 Object Library", version 1.1
4. Now is a good time to compile and make sure the solution is happy, it should look like the following screenshot
# Update the Resources.resx
Update the Resources.resx file to have the image saved earlier
1. Double click the Resources.resx file in solution explorer
2. In the type dropdown, change it from Strings to Images


3. Add your image.  The easiest way to do this is to drag your image (showform.png under the Resources folder) from the solution explorer to the canvas space.  You should now see showform with your image.

4. Add your xml file.  Change the type drop down to Files, and drag customUI into the blank canvas.

# Edit the customUI.xml file.  

This file tells OneNote where your image will be placed, what title, image, and section it is in.   
And we will wire it up to an event, so we can actually do something when the button is clicked.  
1. The ribbon has specific id's associated with the tabs and groups, which you can look up online (too many to list here), but for the sake of this tutorial we will put it on the Home tab, at the far right side of the ribbon (or if you've rearranged, after the mail group).  Once you get the appropriate name, you can always swap those here to put the control in a different place.
2. Here is a copy of the xml:  
``` xml
<?xml version="1.0" encoding="utf-8"?>
<customUI xmlns="http://schemas.microsoft.com/office/2006/01/customui" loadImage="OnGetImage">
<ribbon>
<tabs>
  <tab idMso="TabHome">
    <group id="grpCustomTools" label="Tools" insertAfterMso="GroupMail">
      <button id="btnShowForm" label="Show Form" size="large" onAction="ShowForm" image="showform.png" screentip="Show windows form." />
    </group >
  </tab>
</tabs>
</ribbon>
</customUI>
```  

# Edit the Connect.cs file.  
This is where a lot of the communication code happens between the add-in and the application.
1. Add the "GuidAttribute" and "ProgId" attributes to the class (right above the class declaration).  You will want to generate a guid, and use the namespace and class name for the ProgId.
  - 1.1. This will require the using statement: 
using System.Runtime.InteropServices;

  - 1.2 Example of attributes: 

[GuidAttribute("797efb51-6568-40c2-9564-f60683251281"), ProgId("OneNoteRibbonAddIn.Connect")]

2. Implement the IRibbonExtensibility and IDTExtensibility2 interfaces, and add the using statements.
```C#
Using Extensibility;
Using Microsoft.Office.Core;
```  

3. The code for the rest of the Connect.cs class come from the tutorial online for OneNote 2010, but I will include it here as well (https://code.msdn.microsoft.com/office/CSOneNoteRibbonAddIn-c3547362)
 - 3.1. First let's implement the OnConnection method
1) Create a global class variable called _applicationObject, of type Object
2) Then copy the method variable to the global variable  
```C#  
private object _applicationObject;
        public void OnConnection(object application, ext_ConnectMode connectMode, object addInInst, ref Array custom)
        {
            _applicationObject = application;
        }

```  	
  -3.2. We will now implement the GetCustomUI method, and the rest can be left empty
1) Add return Resources.customUI;
```C#
public string GetCustomUI(string ribbonId)
        {
            return Resources.customUI;
        }
```  
 - 3.3. We will add 2 more methods to the Connect.cs class
 - 
1) public IStream OnGetImage(string imageName)
  a) This is how we will get the image for our ribbon button
2) public void ShowForm(IRibbonControl control)
  a) This is the method that gets called when the user clicks the button on the ribbon in OneNote.  We will wire this up to show our form
3) Here are the complete contents of those methods:
```C#
        // This gets the image for the addin
        public IStream OnGetImage(string imageName)
        {
            MemoryStream stream = new MemoryStream();
            if (imageName == "showform.png")  // should be the same as previouly added image
            {

Resources.showform.Save(stream, ImageFormat.Png);

need to be the same as previously added image in Resources.resx




            }

            return new ReadOnlyIStreamWrapper(stream);
        }

        // This will enable the form to be displayed when clicking the add in button
        public void ShowForm(IRibbonControl control)
        {
            Window context = control.Context as Window;
            if (context != null)
            {
                CWin32WindowWrapper owner = new CWin32WindowWrapper((IntPtr)context.WindowHandle);
                MainForm form = new MainForm(_applicationObject as Application);
                form.ShowDialog(owner);

                form.Dispose();
                form = null;
                context = null;
                owner = null;
            }
            GC.Collect();
            GC.WaitForPendingFinalizers();
            GC.Collect();
        }

```  

4) This file will not currently compile, we need to add some more code first.

12. Let's Edit the CWin32WindowWrapper.cs file
```C#
using System;
using System.Windows.Forms;

namespace OneNoteRibbonAddIn
{
  internal class CWin32WindowWrapper : IWin32Window
  {
      private readonly IntPtr _windowHandle;

      public CWin32WindowWrapper(IntPtr windowHandle)
      {
          _windowHandle = windowHandle;
      }

      public IntPtr Handle
      {
          get { return _windowHandle; }
      }
  }
}
```  		

# Edit the ```ReadOnlyIStreamWrapper.cs```
```C#		
using System;
using System.IO;
using System.Runtime.InteropServices;
using System.Runtime.InteropServices.ComTypes;
using STATSTG = System.Runtime.InteropServices.ComTypes.STATSTG;

namespace OneNoteRibbonAddIn
{
    class ReadOnlyIStreamWrapper : IStream
    {
        private readonly MemoryStream _stream;

        public ReadOnlyIStreamWrapper(MemoryStream stream)
        {
            _stream = stream;
        }

        public void Read(byte[] pv, int cb, IntPtr pcbRead)
        {
            Marshal.WriteInt64(pcbRead, _stream.Read(pv, 0, cb));
        }

        public void Write(byte[] pv, int cb, IntPtr pcbWritten)
        {
            Marshal.WriteInt64(pcbWritten, 0L);
            _stream.Write(pv, 0, cb);
            Marshal.WriteInt64(pcbWritten, cb);
        }

        public void Seek(long dlibMove, int dwOrigin, IntPtr plibNewPosition)
        {
            long num;
            Marshal.WriteInt64(plibNewPosition, _stream.Position);
            switch (dwOrigin)
            {
                case 0:
                    num = dlibMove;
                    break;

                case 1:
                    num = _stream.Position + dlibMove;
                    break;

                case 2:
                    num = _stream.Length + dlibMove;
                    break;

                default:
                    return;
            }
            if ((num >= 0L) && (num < _stream.Length))
            {
                _stream.Position = num;
                Marshal.WriteInt64(plibNewPosition, _stream.Position);
            }
        }

        public void SetSize(long libNewSize)
        {
            _stream.SetLength(libNewSize);
        }

        public void CopyTo(IStream pstm, long cb, IntPtr pcbRead, IntPtr pcbWritten)
        {
            throw new NotSupportedException("ReadOnlyIStreamWrapper does not support CopyTo");
        }

        public void Commit(int grfCommitFlags)
        {
            _stream.Flush();
        }

        public void Revert()
        {
            throw new NotSupportedException("Stream does not support CopyTo");
        }

        public void LockRegion(long libOffset, long cb, int dwLockType)
        {
            throw new NotSupportedException("ReadOnlyIStreamWrapper does not support CopyTo");
        }

        public void UnlockRegion(long libOffset, long cb, int dwLockType)
        {
            throw new NotSupportedException("ReadOnlyIStreamWrapper does not support UnlockRegion");
        }

        public void Stat(out STATSTG pstatstg, int grfStatFlag)
        {
            pstatstg = new STATSTG();
            pstatstg.cbSize = _stream.Length;
            if ((grfStatFlag & 1) == 0)
            {
                pstatstg.pwcsName = _stream.ToString();
            }
        }

        public void Clone(out IStream ppstm)
        {
            ppstm = new ReadOnlyIStreamWrapper(_stream);
        }
    }
}

```  
# Modify MainForm.cs
```C#
[ComVisible(false)]
public partial class MainForm : Form
{
    public MainForm(OneNote.Application oneNoteApp)
    {
        InitializeComponent();
    }
}
```	  			
# Modify the installer package
1. Select the setup project, go to the properties (F4)
  - You can update the properties that make sense, such as Author, Description, Manufacturer
  - Right click the setup file, click Add, then Project Output
    - i. In the Add Project Output Group dialog, select Primary Output, hit OK
    - ii. Right Click on the Primary output from OneNoteRibbonAddIn and go to Properties
2. Change the Register property to vsdrpCOM
  - Add the registry entries (Note: replace <guid> with the guid you generated and added to your attribute in the Connect.cs class, the guid is also always added between squiggly brackets - ex: "{797efb51-6568-40c2-9564-f60683251281}")
  - Right click the setup project, click view, and select Registry
  - Add the following registry keys (*Note: 32 bit on the left, 64 bit on the right)
  ```
1) HKEY_CLASSES_ROOT\AppID\{<guid>} | HKEY_CLASSES_ROOT\Wow6432Node\AppID\{<guid>}
a) Add string DllSurrogate with value ""
2) HKEY_CLASSES_ROOT\CLSID\{<guid>} | HKEY_CLASSES_ROOT\Wow6432Node\CLSID\{<guid>}
a) Add string AppID with value {<guid>}
3) HKEY_CURRENT_USER\Software\Microsoft\Office\OneNote\AddIns\OneNoteRibbonAddIn.Connect
a) Add string Description with value "OneNote 2013 Ribbon Add-In Example"
b) Add string FriendlyName with value "OneNoteRibbonAddIn"
c) Add DWORD LoadBehavior with value 3
4) HKEY_LOCAL_MACHINE\Software\Classes\AppID\{<guid>} | HKEY_LOCAL_MACHINE\Software\Wow6432Node\Classes\AppID\{<guid>}
a) Add string DllSurrogate with value ""
5) HKEY_LOCAL_MACHINE\Software\Classes\CLSID\{<guid>} | HKEY_LOCAL_MACHINE\Software\Wow6432Node\Classes\CLSID\{<guid>}
a) Add string AppID with value {<guid>}
```
3. On all the keys you just added, except the HKEY_CURRENT_USER key (1, 2, 4, and 5 above), open the properties and set the DeleteAtUninstall property to True

# Verification
a. Build the project: Rightclick the setup project and choose Build.
b. Right click your installer project, and click Install, let the installer run
c. If OneNote is open, close it, wait for a few seconds, and then re-open it
d. Note: At this point, if you change anything you don't have to run the installer again, just close OneNote and recompile, then re-open OneNote
e. You should see your form in the ribbon
# Debugging
a. Since the project is a class library, it cannot be started directly.  What you need to do is attach to the dllhost.exe process
i. Start OneNote first
ii. Debug -> Attach to Process…
iii. Browse to the dllhost.exe process, select it, and hit the Attach button
iv. Now you should be able to hit breakpoints, and step through code like normal
# Sanity Check
a. At this point you should be able to see your ribbon icon, with appropriate text, and it should pop open an empty form when you click the button, also you should be able to set a breakpoint in the ShowForm method inside of Connect.cs and get the debugger to break when you click the ribbon button (if you've attached to the process)
3. 18. Code up the form
a. Essentially at this point you can add any code to the windows form that you want, and I will leave this part up to your imagination

# Making changes
a. When you compile, the compiler will complain if OneNote is open.  So, close OneNote, then recompile.  This will also allow your changes to take immediate effect, there is no reason to uninstall / reinstall.
# Example Project
a. I've added this project to GitHub, available here: https://github.com/Fixxzer/OneNoteRibbonAddIn

来自 <https://github.com/Fixxzer/OneNoteRibbonAddIn> 

