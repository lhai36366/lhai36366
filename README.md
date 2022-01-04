### Hi there 👋

<!--
**lhai36366/lhai36366** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
WPF Partial Trust Security
Article
10 minutes to read
In general, Internet applications should be restricted from having direct access to critical system resources, to prevent malicious damage. By default, HTML and client-side scripting languages are not able to access critical system resources. Because Windows Presentation Foundation (WPF) browser-hosted applications can be launched from the browser, they should conform to a similar set of restrictions. To enforce these restrictions, WPF relies on both Code Access Security (CAS) and ClickOnce (see WPF Security Strategy - Platform Security). By default, browser-hosted applications request the Internet zone CAS set of permissions, irrespective of whether they are launched from the Internet, the local intranet, or the local computer. Applications that run with anything less than the full set of permissions are said to be running with partial trust.
WPF provides a wide variety of support to ensure that as much functionality as possible can be used safely in partial trust, and along with CAS, provides additional support for partial trust programming.
This topic contains the following sections:
WPF Feature Partial Trust Support
Partial Trust Programming
Managing Permissions
WPF Feature Partial Trust Support

The following table lists the high-level features of Windows Presentation Foundation (WPF) that are safe to use within the limits of the Internet zone permission set.
Table 1: WPF Features that are Safe in Partial Trust
Feature Area
Feature
General
Browser Window
Site of Origin Access
IsolatedStorage (512KB Limit)
UIAutomation Providers
Commanding
Input Method Editors (IMEs)
Tablet Stylus and Ink
Simulated Drag/Drop using Mouse Capture and Move Events
OpenFileDialog
XAML Deserialization (via XamlReader.Load)
Web Integration
Browser Download Dialog
Top-Level User-Initiated Navigation
mailto:links
Uniform Resource Identifier Parameters
HTTPWebRequest
WPF Content Hosted in an IFRAME
Hosting of Same-Site HTML Pages using Frame
Hosting of Same Site HTML Pages using WebBrowser
Web Services (ASMX)
Web Services (using Windows Communication Foundation)
Scripting
Document Object Model
Visuals
2D and 3D
Animation
Media (Site Of Origin and Cross-Domain)
Imaging/Audio/Video
Reading
FlowDocuments
XPS Documents
Embedded & System Fonts
CFF & TrueType Fonts
Editing
Spell Checking
RichTextBox
Plaintext and Ink Clipboard Support
User-Initiated Paste
Copying Selected Content
Controls
General Controls
This table covers the WPF features at a high level. For more detailed information, the Windows Software Development Kit (SDK) documents the permissions that are required by each member in WPF. Additionally, the following features have more detailed information regarding partial trust execution, including special considerations.
XAML (see XAML Overview (WPF)).
Popups (see System.Windows.Controls.Primitives.Popup).
Drag and Drop (see Drag and Drop Overview).
Clipboard (see System.Windows.Clipboard).
Imaging (see System.Windows.Controls.Image).
Serialization (see XamlReader.Load, XamlWriter.Save).
Open File Dialog Box (see Microsoft.Win32.OpenFileDialog).
The following table outlines the WPF features that are not safe to run within the limits of the Internet zone permission set.
Table 2: WPF Features that are Not Safe in Partial Trust
Feature Area
Feature
General
Window (Application Defined Windows and Dialog Boxes)
SaveFileDialog
File System
Registry Access
Drag and Drop
XAML Serialization (via XamlWriter.Save)
UIAutomation Clients
Source Window Access (HwndHost)
Full Speech Support
Windows Forms Interoperability
Visuals
Bitmap Effects
Image Encoding
Editing
Rich Text Format Clipboard
Full XAML support
Partial Trust Programming

For XBAP applications, code that exceeds the default permission set will have different behavior depending on the security zone. In some cases, the user will receive a warning when they attempt to install it. The user can choose to continue or cancel the installation. The following table describes the behavior of the application for each security zone and what you have to do for the application to receive full trust.
Security Zone
Behavior
Getting Full Trust
Local computer
Automatic full trust
No action is needed.
Intranet and trusted sites
Prompt for full trust
Sign the XBAP with a certificate so that the user sees the source in the prompt.
Internet
Fails with "Trust Not Granted"
Sign the XBAP with a certificate.
Note
The behavior described in the previous table is for full trust XBAPs that do not follow the ClickOnce Trusted Deployment model.
In general, code that may exceed the allowed permissions is likely to be common code that is shared between both standalone and browser-hosted applications. CAS and WPF offer several techniques for managing this scenario.
Detecting Permissions Using CAS

In some situations, it is possible for shared code in library assemblies to be used by both standalone applications and XBAPs. In these cases, code may execute functionality that could require more permissions than the application's awarded permission set allows. Your application can detect whether or not it has a certain permission by using Microsoft .NET Framework security. Specifically, it can test whether it has a specific permission by calling the Demand method on the instance of the desired permission. This is shown in the following example, which has code that queries for whether it has the ability to save a file to the local disk:

Imports System.IO ' File, FileStream, StreamWriter
Imports System.IO.IsolatedStorage ' IsolatedStorageFile
Imports System.Security ' CodeAccesPermission, IsolatedStorageFileStream
Imports System.Security.Permissions ' FileIOPermission, FileIOPermissionAccess
Imports System.Windows ' MessageBox

Namespace SDKSample
    Public Class FileHandling
        Public Sub Save()
            If IsPermissionGranted(New FileIOPermission(FileIOPermissionAccess.Write, "c:\newfile.txt")) Then
                ' Write to local disk
                Using stream As FileStream = File.Create("c:\newfile.txt")
                Using writer As New StreamWriter(stream)
                    writer.WriteLine("I can write to local disk.")
                End Using
                End Using
            Else
                MessageBox.Show("I can't write to local disk.")
            End If
        End Sub

        ' Detect whether or not this application has the requested permission
        Private Function IsPermissionGranted(ByVal requestedPermission As CodeAccessPermission) As Boolean
            Try
                ' Try and get this permission
                requestedPermission.Demand()
                Return True
            Catch
                Return False
            End Try
        End Function



...


    End Class
End Namespace
using System.IO; // File, FileStream, StreamWriter
using System.IO.IsolatedStorage; // IsolatedStorageFile
using System.Security; // CodeAccesPermission, IsolatedStorageFileStream
using System.Security.Permissions; // FileIOPermission, FileIOPermissionAccess
using System.Windows; // MessageBox

namespace SDKSample
{
    public class FileHandling
    {
        public void Save()
        {
            if (IsPermissionGranted(new FileIOPermission(FileIOPermissionAccess.Write, @"c:\newfile.txt")))
            {
                // Write to local disk
                using (FileStream stream = File.Create(@"c:\newfile.txt"))
                using (StreamWriter writer = new StreamWriter(stream))
                {
                    writer.WriteLine("I can write to local disk.");
                }
            }
            else
            {
                MessageBox.Show("I can't write to local disk.");
            }
        }

        // Detect whether or not this application has the requested permission
        bool IsPermissionGranted(CodeAccessPermission requestedPermission)
        {
            try
            {
                // Try and get this permission
                requestedPermission.Demand();
                return true;
            }
            catch
            {
                return false;
            }
        }



...


    }
}
If an application does not have the desired permission, the call to Demand will throw a security exception. Otherwise, the permission has been granted. IsPermissionGranted encapsulates this behavior and returns true or false as appropriate.
Graceful Degradation of Functionality

Being able to detect whether code has the permission to do what it needs to do is interesting for code that can be executed from different zones. While detecting the zone is one thing, it is far better to provide an alternative for the user, if possible. For example, a full trust application typically enables users to create files anywhere they want, while a partial trust application can only create files in isolated storage. If the code to create a file exists in an assembly that is shared by both full trust (standalone) applications and partial trust (browser-hosted) applications, and both applications want users to be able to create files, the shared code should detect whether it is running in partial or full trust before creating a file in the appropriate location. The following code demonstrates both.

Imports System.IO ' File, FileStream, StreamWriter
Imports System.IO.IsolatedStorage ' IsolatedStorageFile
Imports System.Security ' CodeAccesPermission
Imports System.Security.Permissions ' FileIOPermission, FileIOPermissionAccess
Imports System.Windows ' MessageBox

Namespace SDKSample
    Public Class FileHandlingGraceful
        Public Sub Save()
            If IsPermissionGranted(New FileIOPermission(FileIOPermissionAccess.Write, "c:\newfile.txt")) Then
                ' Write to local disk
                Using stream As FileStream =tênthietbi:<hailuu(GT-19195)>phienban.android<4.4.2>capdovaloibaomat<2016-05-01>phienban.baseband<19195xxscQA3> File.Create("c:\newfile.txt")
                Using writer As New StreamWriter(stream)
                    writer.WriteLine("I can write to local disk.")
                End Using
                End Using
            Else
                ' Persist application-scope property to 
                ' isolated storage
                Dim storage As IsolatedStorageFile =phien.kernel<3.4.0-8086469>#¿¿¿¿¿#(#DPI@SWHC3707#1)¿¿¿¿¿WE¿JAN<4 20:32:33 KST2017¿¿¿¿¿>#sohieubantao#¿<kot49h.19195xxscQA3¿¿¿¿>#<trangthai.SE.cho"ANDROID"ENFORCING¿¿¿¿¿#<Sepf-9T-19195-4.4.2-0054>#¿¿¿¿¿-web¿an#<04 20:31:30 2017>¿¿¿¿¿#secureboo¿¿¿atus<¿#"type:SAMSUNG"¿¿¿¿¿#>| <?xml version="1.0" encoding="UTF-8"?>
<feed xmlns="http://www.w3.org/2005/Atom" xmlns:media="http://search.yahoo.com/mrss/" xml:lang="en-US">
  <id>tag:github.com,2008:/Hailuu3333</id>
  <link type="text/html" rel="alternate" href="https://github.com/Hailuu3333"/>
  <link type="application/atom+xml" rel="self" href="https://github.com/Hailuu3333.private.atom?token=AWURDBH7ASDDKL3TWM5OQS57363BS"/>
  <title>Private Feed for Hailuu3333</title>
  <updated>2022-01-03T17:58:47Z</updated>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562318704</id>
    <published>2022-01-03T17:58:47Z</published>
    <updated>2022-01-03T17:58:47Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/e746e8910e...5e157d1ff0"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:58:47Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/5e157d1ff014dcb8f80f421cdd001bdec6fba990&quot; rel=&quot;noreferrer&quot;&gt;5e157d1&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update all browsers versions for PerformanceResourceTiming API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090544464&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14304&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14304/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14304&quot; rel=&quot;noreferrer&quot;&gt;#14304&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562301951</id>
    <published>2022-01-03T17:57:13Z</published>
    <updated>2022-01-03T17:57:13Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/ba5bbe1657...e746e8910e"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:57:13Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/e746e8910e7defdbf34cc9ef0e05965debe30f4f&quot; rel=&quot;noreferrer&quot;&gt;e746e89&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update WebView versions for PerformanceObserverEntryList API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090457391&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14301&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14301/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14301&quot; rel=&quot;noreferrer&quot;&gt;#14301&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562295899</id>
    <published>2022-01-03T17:56:39Z</published>
    <updated>2022-01-03T17:56:39Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/f52107082e...ba5bbe1657"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:56:39Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/ba5bbe16577941c22b7e69856f1dcfacbb5a3328&quot; rel=&quot;noreferrer&quot;&gt;ba5bbe1&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox versions for PerformanceMeasure API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090448000&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14299&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14299/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14299&quot; rel=&quot;noreferrer&quot;&gt;#14299&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562286119</id>
    <published>2022-01-03T17:55:43Z</published>
    <updated>2022-01-03T17:55:43Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/04b6bd0c15...f52107082e"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:55:43Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/f52107082e4f9ef8ac6e98b3f7dfd2795309d824&quot; rel=&quot;noreferrer&quot;&gt;f521070&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update all browsers versions for PerformanceMark API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090446499&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14298&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14298/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14298&quot; rel=&quot;noreferrer&quot;&gt;#14298&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562263337</id>
    <published>2022-01-03T17:53:34Z</published>
    <updated>2022-01-03T17:53:34Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/75674de739...04b6bd0c15"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:53:34Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/04b6bd0c157e737639b1ae7f94f30fa2168effa3&quot; rel=&quot;noreferrer&quot;&gt;04b6bd0&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for PerformanceEventTiming API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090443593&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14296&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14296/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14296&quot; rel=&quot;noreferrer&quot;&gt;#14296&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562241843</id>
    <published>2022-01-03T17:51:36Z</published>
    <updated>2022-01-03T17:51:36Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/29a6974a75...75674de739"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:51:36Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/75674de739775f3dd4cddef82d09b98c9a058cf9&quot; rel=&quot;noreferrer&quot;&gt;75674de&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox versions for api.PerformanceEventTiming.toJSON (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090445235&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14297&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14297/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14297&quot; rel=&quot;noreferrer&quot;&gt;#14297&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562237548</id>
    <published>2022-01-03T17:51:13Z</published>
    <updated>2022-01-03T17:51:13Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/bf39eba737...29a6974a75"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:51:13Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/29a6974a75547ca3bac6b24878bb0967c864f41a&quot; rel=&quot;noreferrer&quot;&gt;29a6974&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update WebView versions for api.OffscreenCanvasRenderingContext2D.can…
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562232818</id>
    <published>2022-01-03T17:50:48Z</published>
    <updated>2022-01-03T17:50:48Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/3eeeac9f61...bf39eba737"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:50:48Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/bf39eba7371b943fbe862dff0af4d0406729f94f&quot; rel=&quot;noreferrer&quot;&gt;bf39eba&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for OTPCredential API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090403975&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14290&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14290/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14290&quot; rel=&quot;noreferrer&quot;&gt;#14290&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19562188285</id>
    <published>2022-01-03T17:46:40Z</published>
    <updated>2022-01-03T17:46:40Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/4bff6425fa...3eeeac9f61"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:46:40Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/3eeeac9f61d756098fe70aeb24f502e6017750f4&quot; rel=&quot;noreferrer&quot;&gt;3eeeac9&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update all browsers versions for CanvasPattern API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089195844&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14202&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14202/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14202&quot; rel=&quot;noreferrer&quot;&gt;#14202&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561882855</id>
    <published>2022-01-03T17:19:44Z</published>
    <updated>2022-01-03T17:19:44Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/a74e1ff439...4bff6425fa"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:19:44Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/4bff6425faac6cbb58b15fc609993d960edd6827&quot; rel=&quot;noreferrer&quot;&gt;4bff642&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for NDEF APIs (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090332102&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14283&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14283/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14283&quot; rel=&quot;noreferrer&quot;&gt;#14283&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561829380</id>
    <published>2022-01-03T17:15:22Z</published>
    <updated>2022-01-03T17:15:22Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/edf64e0c97...a74e1ff439"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:15:22Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/a74e1ff4395fb92a0844164f5463d64632e7c2ca&quot; rel=&quot;noreferrer&quot;&gt;a74e1ff&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for MessageChannel API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090079194&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14272&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14272/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14272&quot; rel=&quot;noreferrer&quot;&gt;#14272&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561819576</id>
    <published>2022-01-03T17:14:35Z</published>
    <updated>2022-01-03T17:14:35Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/ebbed37e94...edf64e0c97"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:14:35Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/edf64e0c97fdfae1dc818946f8227ab94724d056&quot; rel=&quot;noreferrer&quot;&gt;edf64e0&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox Android versions for api.MediaStreamAudioSourceNode.me…
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561801160</id>
    <published>2022-01-03T17:13:04Z</published>
    <updated>2022-01-03T17:13:04Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/20fa4010c1...ebbed37e94"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:13:04Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/ebbed37e948956bcbc9022f49ef6f33abca74e46&quot; rel=&quot;noreferrer&quot;&gt;ebbed37&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update WebView versions for MediaKeyMessageEvent API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090075095&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14268&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14268/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14268&quot; rel=&quot;noreferrer&quot;&gt;#14268&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561788980</id>
    <published>2022-01-03T17:12:05Z</published>
    <updated>2022-01-03T17:12:05Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/d9a8229dc0...20fa4010c1"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:12:05Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/20fa4010c12cd5a0e74b1eb8cbea4d3afd89f264&quot; rel=&quot;noreferrer&quot;&gt;20fa401&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox Android versions for api.MediaElementAudioSourceNode.m…
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561782081</id>
    <published>2022-01-03T17:11:33Z</published>
    <updated>2022-01-03T17:11:33Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/219bae08f0...d9a8229dc0"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:11:33Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/d9a8229dc08bed161e100d25ca154375f0333fb3&quot; rel=&quot;noreferrer&quot;&gt;d9a8229&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for MediaDeviceInfo API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090073619&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14266&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14266/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14266&quot; rel=&quot;noreferrer&quot;&gt;#14266&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561757800</id>
    <published>2022-01-03T17:09:36Z</published>
    <updated>2022-01-03T17:09:36Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/77f4c09ac6...219bae08f0"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:09:36Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/219bae08f0ff3917c23d60cfa18d4701d8153d26&quot; rel=&quot;noreferrer&quot;&gt;219bae0&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Edge versions for KeyframeEffect API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1090071498&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14265&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14265/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14265&quot; rel=&quot;noreferrer&quot;&gt;#14265&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561683563</id>
    <published>2022-01-03T17:03:51Z</published>
    <updated>2022-01-03T17:03:51Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/8d3ddcf273...77f4c09ac6"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:03:51Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/77f4c09ac6f380764dc5af24fb460e2d8d2303e4&quot; rel=&quot;noreferrer&quot;&gt;77f4c09&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for IdleDetector API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089631424&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14258&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14258/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14258&quot; rel=&quot;noreferrer&quot;&gt;#14258&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561641770</id>
    <published>2022-01-03T17:00:42Z</published>
    <updated>2022-01-03T17:00:42Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/43b1c984e3...8d3ddcf273"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T17:00:42Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/8d3ddcf273becc09f9b9b36139326ca26eee4589&quot; rel=&quot;noreferrer&quot;&gt;8d3ddcf&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for HTMLQuoteElement API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089623277&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14256&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14256/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14256&quot; rel=&quot;noreferrer&quot;&gt;#14256&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561618310</id>
    <published>2022-01-03T16:58:55Z</published>
    <updated>2022-01-03T16:58:55Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/e0b6775007...43b1c984e3"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:58:55Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/43b1c984e3a15c611b0abc314cc8b0046664d3b5&quot; rel=&quot;noreferrer&quot;&gt;43b1c98&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update all browsers versions for HTMLFormControlsCollection API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089618163&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14253&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14253/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14253&quot; rel=&quot;noreferrer&quot;&gt;#14253&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561587537</id>
    <published>2022-01-03T16:56:25Z</published>
    <updated>2022-01-03T16:56:25Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/69f98959d0...e0b6775007"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:56:25Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/e0b67750076ad0d93393ccb085d1dc54d773253e&quot; rel=&quot;noreferrer&quot;&gt;e0b6775&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for api.HTMLElement.outerText (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089616978&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14252&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14252/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14252&quot; rel=&quot;noreferrer&quot;&gt;#14252&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561489898</id>
    <published>2022-01-03T16:48:36Z</published>
    <updated>2022-01-03T16:48:36Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/528d168175...69f98959d0"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:48:36Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/69f98959d01049d80574d185e8e74b11bacad813&quot; rel=&quot;noreferrer&quot;&gt;69f9895&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update WebView versions for HTMLCanvasElement API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089594658&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14251&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14251/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14251&quot; rel=&quot;noreferrer&quot;&gt;#14251&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561478464</id>
    <published>2022-01-03T16:47:42Z</published>
    <updated>2022-01-03T16:47:42Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/1c83c46f3b...528d168175"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:47:42Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/528d168175ca12effcb92663441b0bb5dba86c36&quot; rel=&quot;noreferrer&quot;&gt;528d168&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update WebView versions for GamepadHapticActuator API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089591518&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14250&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14250/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14250&quot; rel=&quot;noreferrer&quot;&gt;#14250&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561470552</id>
    <published>2022-01-03T16:47:04Z</published>
    <updated>2022-01-03T16:47:04Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/8988fadf71...1c83c46f3b"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:47:04Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/1c83c46f3b35669320196ee8ed4c058e6e27f81b&quot; rel=&quot;noreferrer&quot;&gt;1c83c46&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox Android versions for FormDataEvent API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089591110&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14249&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14249/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14249&quot; rel=&quot;noreferrer&quot;&gt;#14249&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561432212</id>
    <published>2022-01-03T16:44:01Z</published>
    <updated>2022-01-03T16:44:01Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/aee84e62b4...8988fadf71"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:44:01Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/8988fadf71141dc430fee7270277952475021227&quot; rel=&quot;noreferrer&quot;&gt;8988fad&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for FormData API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089590697&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14248&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14248/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14248&quot; rel=&quot;noreferrer&quot;&gt;#14248&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561367603</id>
    <published>2022-01-03T16:38:51Z</published>
    <updated>2022-01-03T16:38:51Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/ac60c669b9...aee84e62b4"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:38:51Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/aee84e62b43d1ef88fa59b157870bd2489d6586d&quot; rel=&quot;noreferrer&quot;&gt;aee84e6&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox Android versions for api.FetchEvent.handled (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089588080&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14245&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14245/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14245&quot; rel=&quot;noreferrer&quot;&gt;#14245&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561363029</id>
    <published>2022-01-03T16:38:29Z</published>
    <updated>2022-01-03T16:38:29Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/6e0eb31096...ac60c669b9"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:38:29Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/ac60c669b9fd4329e9d466de32b45535970748de&quot; rel=&quot;noreferrer&quot;&gt;ac60c66&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Chromium versions for FederatedCredential API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089587721&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14244&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14244/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14244&quot; rel=&quot;noreferrer&quot;&gt;#14244&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561354449</id>
    <published>2022-01-03T16:37:48Z</published>
    <updated>2022-01-03T16:37:48Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/625928c8b1...6e0eb31096"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:37:48Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/6e0eb31096b675addd01f2a4c7086881cf77b19e&quot; rel=&quot;noreferrer&quot;&gt;6e0eb31&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox versions for ExtendableMessageEvent API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089586405&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14243&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14243/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14243&quot; rel=&quot;noreferrer&quot;&gt;#14243&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561344598</id>
    <published>2022-01-03T16:37:00Z</published>
    <updated>2022-01-03T16:37:00Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/5f702b8587...625928c8b1"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:37:00Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/625928c8b125e9d7d4a86fa8c4e87ec3ff3cb69b&quot; rel=&quot;noreferrer&quot;&gt;625928c&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update WebView versions for ExtendableCookieChangeEvent API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089583771&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14242&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14242/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14242&quot; rel=&quot;noreferrer&quot;&gt;#14242&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561318915</id>
    <published>2022-01-03T16:35:01Z</published>
    <updated>2022-01-03T16:35:01Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/e68f72ae70...5f702b8587"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:35:01Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/5f702b8587e131920441b60a96e28b6b7324efe7&quot; rel=&quot;noreferrer&quot;&gt;5f702b8&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update Firefox versions for api.EventSource.EventSource (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089583044&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14241&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14241/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14241&quot; rel=&quot;noreferrer&quot;&gt;#14241&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
  <entry>
    <id>tag:github.com,2008:PushEvent/19561277837</id>
    <published>2022-01-03T16:31:50Z</published>
    <updated>2022-01-03T16:31:50Z</updated>
    <link type="text/html" rel="alternate" href="https://github.com/mdn/browser-compat-data/compare/31a42b62de...e68f72ae70"/>
    <title type="html">foolip pushed to main in mdn/browser-compat-data</title>
    <author>
      <name>foolip</name>
      <uri>https://github.com/foolip</uri>
    </author>
    <media:thumbnail height="30" width="30" url="https://avatars.githubusercontent.com/u/498917?s=30&amp;v=4"/>
    <content type="html">&lt;div class=&quot;push&quot;&gt;&lt;div class=&quot;body&quot;&gt;
&lt;!-- push --&gt;
&lt;div class=&quot;d-flex flex-items-baseline border-bottom color-border-muted py-3&quot;&gt;
    &lt;span class=&quot;mr-2&quot;&gt;&lt;a class=&quot;d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;avatar avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/498917?s=64&amp;amp;v=4&quot; width=&quot;32&quot; height=&quot;32&quot; alt=&quot;@foolip&quot;&gt;&lt;/a&gt;&lt;/span&gt;
  &lt;div class=&quot;d-flex flex-column width-full&quot;&gt;
    &lt;div class=&quot;&quot;&gt;
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/foolip&quot; rel=&quot;noreferrer&quot;&gt;foolip&lt;/a&gt;
      
      pushed to
      &lt;a class=&quot;Link--primary no-underline wb-break-all text-bold d-inline-block&quot; href=&quot;/mdn/browser-compat-data&quot; rel=&quot;noreferrer&quot;&gt;mdn/browser-compat-data&lt;/a&gt;
        &lt;span class=&quot;color-fg-muted no-wrap f6 ml-1&quot;&gt;
          &lt;relative-time datetime=&quot;2022-01-03T16:31:50Z&quot; class=&quot;no-wrap&quot;&gt;Jan 3, 2022&lt;/relative-time&gt;
        &lt;/span&gt;

        &lt;div class=&quot;Box p-3 mt-2 &quot;&gt;
          &lt;span&gt;1 commit to&lt;/span&gt;
          &lt;a class=&quot;branch-name&quot; href=&quot;/mdn/browser-compat-data/tree/main&quot; rel=&quot;noreferrer&quot;&gt;main&lt;/a&gt;

          &lt;div class=&quot;commits &quot;&gt;
            &lt;ul class=&quot;list-style-none&quot;&gt;
                &lt;li class=&quot;d-flex flex-items-baseline&quot;&gt;
                  &lt;span title=&quot;queengooborg&quot;&gt;
                    &lt;a class=&quot;d-inline-block&quot; href=&quot;/queengooborg&quot; rel=&quot;noreferrer&quot;&gt;&lt;img class=&quot;mr-1 avatar-user&quot; src=&quot;https://avatars.githubusercontent.com/u/5179191?s=32&amp;amp;v=4&quot; width=&quot;16&quot; height=&quot;16&quot; alt=&quot;@queengooborg&quot;&gt;&lt;/a&gt;
                  &lt;/span&gt;
                  &lt;code&gt;&lt;a class=&quot;mr-1&quot; href=&quot;/mdn/browser-compat-data/commit/e68f72ae70c99bb9c5be7489cce03c6e4d44c8f7&quot; rel=&quot;noreferrer&quot;&gt;e68f72a&lt;/a&gt;&lt;/code&gt;
                  &lt;div class=&quot;dashboard-break-word lh-condensed&quot;&gt;
                    &lt;blockquote&gt;
                      Update all browsers versions for Event API (&lt;a class=&quot;issue-link js-issue-link&quot; data-error-text=&quot;Failed to load title&quot; data-id=&quot;1089581729&quot; data-permission-text=&quot;Title is private&quot; data-url=&quot;https://github.com/mdn/browser-compat-data/issues/14239&quot; data-hovercard-type=&quot;pull_request&quot; data-hovercard-url=&quot;/mdn/browser-compat-data/pull/14239/hovercard&quot; href=&quot;https://github.com/mdn/browser-compat-data/pull/14239&quot; rel=&quot;noreferrer&quot;&gt;#14239&lt;/a&gt;)
                    &lt;/blockquote&gt;
                  &lt;/div&gt;
                &lt;/li&gt;


            &lt;/ul&gt;
          &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/div&gt;
&lt;/div&gt;&lt;/div&gt;</content>
  </entry>
</feed> IsolatedStorageFile.GetUserStoreForApplication()
                Using stream As New IsolatedStorageFileStream("newfile.txt", FileMode.Create, storage)
                Using writer As New StreamWriter(stream)
                    writer.WriteLine("I can write to Isolated Storage")
                End Using
                End Using
            End If
        End Sub

        ' Detect whether or not this application has the requested permission
        Private Function IsPermissionGranted(ByVal requestedPermission As CodeAccessPermission) As Boolean
            Try
                ' Try and get this permission
                requestedPermission.Demand()
                Return True
            Catch
                Return False
            End Try
        End Function



...


    End Class
End Namespace
using System.IO; // File, FileStream, StreamWriter
using System.IO.IsolatedStorage; // IsolatedStorageFile
using System.Security; // CodeAccesPermission
using System.Security.Permissions; // FileIOPermission, FileIOPermissionAccess
using System.Windows; // MessageBox

namespace SDKSample
{
    public class FileHandlingGraceful
    {
        public void Save()
        {
            if (IsPermissionGranted(new FileIOPermission(FileIOPermissionAccess.Write, @"c:\newfile.txt")))
            {
                // Write to local disk
                using (FileStream stream = File.Create(@"c:\newfile.txt"))
                using (StreamWriter writer = new StreamWriter(stream))
                {
                    writer.WriteLine("I can write to local disk.");
                }
            }
            else
            {
                // Persist application-scope property to 
                // isolated storage
                IsolatedStorageFile storage = IsolatedStorageFile.GetUserStoreForApplication();
                using (IsolatedStorageFileStream stream = 
                    new IsolatedStorageFileStream("newfile.txt", FileMode.Create, storage))
                using (StreamWriter writer = new StreamWriter(stream))
                {
                    writer.WriteLine("I can write to Isolated Storage");
                }
            }
        }

        // Detect whether or not this application has the requested permission
        bool IsPermissionGranted(CodeAccessPermission requestedPermission)
        {
            try
            {
                // Try and get this permission
                requestedPermission.Demand();
                return true;
            }
            catch
            {
                return false;
            }
        }



...


    }
}
In many cases, you should be able to find a partial trust alternative.
In a controlled environment, such as an intranet, custom managed frameworks can be installed across the client base into the global assembly cache (GAC). These libraries can execute code that requires full trust, and be referenced from applications that are only allowed partial trust by using AllowPartiallyTrustedCallersAttribute (for more information, see Security (WPF) and WPF Security Strategy - Platform Security).
Browser Host Detection

Using CAS to check for permissions is a suitable technique when you need to check on a per-permission basis. Although, this technique depends on catching exceptions as a part of normal processing, which is not recommended in general and can have performance issues. Instead, if your XAML browser application (XBAP) only runs within the Internet zone sandbox, you can use the BrowserInteropHelper.IsBrowserHosted property, which returns true for XAML browser applications (XBAPs).
Note
IsBrowserHosted only distinguishes whether an application is running in a browser, not which set of permissions an application is running with.
Managing Permissions

By default, XBAPs run with partial trust (default Internet zone permission set). However, depending on the requirements of the application, it is possible to change the set of permissions from the default. For example, if an XBAPs is launched from a local intranet, it can take advantage of an increased permission set, which is shown in the following table.
Table 3: LocalIntranet and Internet Permissions
Permission
Attribute
LocalIntranet
Internet
DNS
Access DNS servers
Yes
No
Environment Variables
Read
Yes
No
File Dialogs
Open
Yes
Yes
File Dialogs
Unrestricted
Yes
No
Isolated Storage
Assembly isolation by user
Yes
No
Isolated Storage
Unknown isolation
Yes
Yes
Isolated Storage
Unlimited user quota
Yes
No
Media
Safe audio, video, and images
Yes
Yes
Printing
Default printing
Yes
No
Printing
Safe printing
Yes
Yes
Reflection
Emit
Yes
No
Security
Managed code execution
Yes
Yes
Security
Assert granted permissions
Yes
No
User Interface
Unrestricted
Yes
No
User Interface
Safe top level windows
Yes
Yes
User Interface
Own Clipboard
Yes
Yes
Web Browser
Safe frame navigation to HTML
Yes
Yes
Note
Cut and Paste is only allowed in partial trust when user initiated.
If you need to increase permissions, you need to change the project settings and the ClickOnce application manifest. For more information, see WPF XAML Browser Applications Overview. The following documents may also be helpful.
Mage.exe (Manifest Generation and Editing Tool).
MageUI.exe (Manifest Generation and Editing Tool, Graphical Client).
Securing ClickOnce Applications.
If your XBAP requires full trust, you can use the same tools to increase the requested permissions. Although an XBAP will only receive full trust if it is installed on and launched from the local computer, the intranet, or from a URL that is listed in the browser's trusted or allowed sites. If the application is installed from the intranet or a trusted site, the user will receive the standard ClickOnce prompt notifying them of the elevated permissions. The user can choose to continue or cancel the installation. 
Alternatively, you can use the ClickOnce Trusted Deployment model for full trust deployment from any security zone. For more information, see Trusted Application Deployment Overview and Security (WPF).
See Also

Concepts
Security (WPF)
WPF Security Strategy - Platform Security
WPF Security Strategy - Security Engineering

Theme
Previous Version Docs
Blog
Contribute
Privacy & Cookies
Terms of Use
Trademarks
© Microsoft 2021
