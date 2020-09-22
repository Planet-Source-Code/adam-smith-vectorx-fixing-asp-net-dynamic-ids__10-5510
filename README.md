<div align="center">

## Fixing ASP\.NET Dynamic IDs


</div>

### Description

Renders asp.net server ids on the clientside, so that you can find them with getElementById.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Adam Smith \(VectorX\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/adam-smith-vectorx.md)
**Level**          |Intermediate
**User Rating**    |4.7 (14 globes from 3 users)
**Compatibility**  |C\#, ASP\.NET
**Category**       |[Internet/ Browsers/ HTML](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/internet-browsers-html__10-9.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/adam-smith-vectorx-fixing-asp-net-dynamic-ids__10-5510/archive/master.zip)





### Source Code

At my work we use asp.net to build alot of our projects. Since the revolution of AJAX, mixing clientside and serverside has become a real pain. The main issue is ASP.NET dynamically creates Control ID names when the page is rendered. It does this to make sure IDs are unique. So if you want to use javascript getElementById('id name') forget about it. ASP.NET dynamically renders control ids like id = 'ctl001_ControlName_Blah_Blah' when you really just wanted the asp.net property name id='ControlName'.
Below I wrote a code to fix this issue so you dont have to worry about it anymore. Your names will be exactly the same whether serverside or clientside.
There are two methods below needed to render ids. The first method IterateThroughControls(ControlCollection _Parent) uses recursion to gather up a list of controls ids from the controlcollection of the page. It builds a javascript string of var ID = ClientID; which in turn maps the actual id with the dynamic one, so that you can find it with document.getElementById(ControlID). You will need to remove the single quotes in getElementById because you are now using it as a string variable. The second method overrides the page.OnPreRender() to render the javascript with Page.ClientScript.RegisterClientScriptBlock().
  /// <summary>Loops through all controls in a webpage using recursion.</summary>
  /// <PARAM name="_Parent">The page control collection.</PARAM>
  /// <returns>Returns a javascript string of all matching ids.</returns>
  protected string IterateThroughControls(ControlCollection _Parent)
  {
    StringBuilder s = new StringBuilder();
    foreach (Control c in _Parent)
    {
      if (!string.IsNullOrEmpty(c.ID) && !string.IsNullOrEmpty(c.ClientID))
      {
        s.AppendFormat("var {0} = '{1}';\n", c.ID, c.ClientID);
      }
      if (c.Controls.Count > 0) { s.Append(IterateThroughControls(c.Controls)); }
    }
    return s.ToString();
  }
  /// <summary>On Page PreRender,</summary>
  /// <PARAM name="e">EventArgs.</PARAM>
  protected override void OnPreRender(EventArgs e)
  {
    //Load all control ids----------------------------------------
    StringBuilder s = new StringBuilder();
    s.AppendFormat("&#60;script language=\"javascript\">\n{0}&#60;/script>", IterateThroughControls(Page.Controls));
    Page.ClientScript.RegisterClientScriptBlock(this.GetType(), "ControlIDS", s.ToString());
    base.OnPreRender(e);
    //------------------------------------------------------------
  }
To find your control just use its real id like so:
document.getElementById(ControlName).value

