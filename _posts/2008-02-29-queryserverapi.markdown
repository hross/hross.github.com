---
layout: post
title: Querying the Portal Database using the Server API
categories: [webcenter]
---

I love the [ALUI Server API][1]. It's robust, fairly easy to use, cross-platform, and powerful. Unfortunately, it still has its limitations. One of the biggest limitations is the same limitation that plagues most object models built on top of a database layer: its inability to be a database. 

No matter how brilliant the design of the object hierarchy, there will always be situations where running a SQL query to get information would be a lot simpler. Recently, I ran into one of those situations and was lucky enough to know how to circumvent the "rules". What follows is an analysis of that situation.

As usual, this post has me thinking I should just put "**this is not supported**" in my blog description.

### The Problem

I was recently asked, "How can I get a list of all the portlets currently in use in my portal?" In other words, could I come up with a table of pages, their parent communities, and the portlets on them for the entire portal.

Using the server API, creating such a table is possible via the following logic:

1.  Loop through all communities in the portal 
2.  For each community, get a list of pages 
3.  For each page, get a list of portlets. 
4.  Look up each portlet name and ID. 

While this is certainly do-able, you don't need [big O notation][2] to see the embedded for/each statements, large volume of data, and potential for this code to chew up a ton of CPU cycles and make quite a few database queries. In a production environment, this just doesn't seem feasible.

But oh, if I only had access to the database. I could write a complex query that would join a few tables together and give me what I want. One simple SQL statement. Here is the statement I would write to produce the described table:

    SELECT pagegadgets.GADGETID, gadgets.NAME AS GADGETNAME,
    communities.NAME AS COMMUNITYNAME, pages.NAME AS PAGENAME FROM 
    PTPAGEGADGETS pagegadgets LEFT JOIN PTPAGES pages 
        ON pagegadgets.PAGEID=pages.OBJECTID 
    INNER JOIN PTGADGETS gadgets 
        ON pagegadgets.GADGETID=gadgets.OBJECTID 
    INNER JOIN PTCOMMUNITIES communities 
        ON pages.FOLDERID = communities.FOLDERID 
    ORDER BY gadgets.NAME, communities.NAME, pages.NAME ASC

Some of you are probably thinking, "How about I just create a remote portlet that directly connects to the portal database?", which you can do, and some customers have. However, you lose the ability to combine this table with other API code, lose the portable nature of Sever API libraries, and the cross platform capability to execute the query.

How about I show you a way to use the Server API instead?

### Casting to an Internal Session

It turns out that this process is much easier than you think. The first part involves getting an internal session object. An internal session is simply a back end class that we aren't expected to use. It provides a lot of goodies that aren't available to normal server API IPTSession objects. To get an internal session, we simply need to know about it. This means the following imports:

**import com.plumtree.server.impl.core.PTSession;**

**import com.plumtree.server.impl.core.InternalSession;**

and casting an IPTSession object like so:

**InternalSession iSession = ((PTSession) session).GetInternalSession();**

Congratulations. You have an internal session object. Intellisense will show you all kinds of undocumented goodies related to this object. Given the post topic, today we are mainly interested in the database querying ability...

### Running a Query

I could write something long and witty to explain the rest of the code, but it hardly seems necessary. Here is the entire source (Java) for a tag which does as described. The SQL is hard coded into the example:

    package com.bea.services.tags;
    
    import com.plumtree.openkernel.db.IOKDBCursor;
    import com.plumtree.openkernel.db.IOKDBResultSet;
    import com.plumtree.openkernel.db.IOKDBRow;
    import com.plumtree.portaluiinfrastructure.tags.ATag;
    import com.plumtree.portaluiinfrastructure.tags.TagType;
    import com.plumtree.portaluiinfrastructure.tags.metadata.*;
    import com.plumtree.server.*;
    import com.plumtree.server.impl.core.InternalSession;
    import com.plumtree.server.impl.core.PTSession;
    import com.plumtree.taskapi.portalui.TaskAPIUIUser;
    import com.plumtree.uiinfrastructure.activityspace.AActivitySpace;
    import com.plumtree.xpshared.htmlconstructs.PTStyleClass;
    import com.plumtree.xpshared.htmlelements.*;
    
    public class PortletLocationTag extends ATag {
    
        public static final ITagMetaData TAG = new TagMetaData("portletlist",
                "This tag lists portlet locations.");
    
        public static final OptionalTagAttribute PORTLETID = new OptionalTagAttribute(
                "portletId", "A specific portlet ID to query by.",
                AttributeType.STRING, "");
        
        public static final OptionalTagAttribute MAXROWS = new OptionalTagAttribute(
                "maxRows", "The maximum number of rows in the query.",
                AttributeType.STRING, "500");
    
        private static final String PORTLET_LOCATION_QUERY = 
            "SELECT pagegadgets.GADGETID, gadgets.NAME AS GADGETNAME, "
                + "communities.NAME AS COMMUNITYNAME, "
                + "pages.NAME AS PAGENAME FROM "
                + "PTPAGEGADGETS pagegadgets "
                + "LEFT JOIN PTPAGES pages ON "
                + "pagegadgets.PAGEID=pages.OBJECTID "
                + "INNER JOIN PTGADGETS gadgets ON "
                + "pagegadgets.GADGETID=gadgets.OBJECTID "
                + "INNER JOIN PTCOMMUNITIES communities ON "
                + "pages.FOLDERID = communities.FOLDERID ";
        
        private static final String WHERE_PORTLET_ID = "WHERE gadgets.OBJECTID = ";
        private static final String ORDER_BY_GADGET = " ORDER BY gadgets.NAME, communities.NAME, pages.NAME ASC";
        private static final String ORDER_BY_COMMUNITY = " ORDER BY communities.NAME, pages.NAME ASC";
    
        public ATag Create() {
            return new PortletLocationTag();
        }
    
        public TagType GetTagType() {
            return TagType.SIMPLE;
        }
    
        public HTMLElement DisplayTag() {
    
            if (!hasAdminAccess()) {
                return null;
            } // they don't have access
    
            // create a table for our result set
            HTMLTable result = new HTMLTable();
            result.SetWidth(CommonHTMLStrings.ONE_HUNDRED_PERCENT);
            result.SetBorder(CommonHTMLStrings.ZERO);
            result.SetCellPadding(CommonHTMLStrings.ONE);
            result.SetCellSpacing(CommonHTMLStrings.ONE);
    
            // get the user session
            IPTSession session = getSession();
            InternalSession iSession = ((PTSession) session).GetInternalSession();
    
            // run the query as a cursor
            String query = PORTLET_LOCATION_QUERY;
            
            // build the query based on tag options
            if (getPortletId() &gt; 0) {
                query += WHERE_PORTLET_ID + getPortletId() + ORDER_BY_COMMUNITY;
            } else {
                query += ORDER_BY_GADGET;
            }
            
            // open the cursor and run it
            IOKDBCursor cursor = iSession.CreateCursor(query);
            int maxRows = getMaxRows();
            IOKDBResultSet results = cursor.Open(maxRows);
            
            // build the table of results
            HTMLTableRow tableRow = new HTMLTableRow();
            tableRow.SetStyleClass(PTStyleClass.LIST_SORT_HEADER_BG);
    
            HTMLTableCell tableCell = new HTMLTableCell();
            tableCell.SetVAlign(CommonHTMLStrings.MIDDLE);
            tableCell.SetStyleClass(PTStyleClass.LIST_SORT_HEADER);
            tableCell.AddInnerHTMLString("**Portlet Name**");
            tableRow.AddInnerHTMLElement(tableCell);
            
            tableCell = new HTMLTableCell();
            tableCell.SetVAlign(CommonHTMLStrings.MIDDLE);
            tableCell.SetStyleClass(PTStyleClass.LIST_SORT_HEADER);
            tableCell.AddInnerHTMLString("**Community Name**");
            tableRow.AddInnerHTMLElement(tableCell);
            
            tableCell = new HTMLTableCell();
            tableCell.SetVAlign(CommonHTMLStrings.MIDDLE);
            tableCell.SetStyleClass(PTStyleClass.LIST_SORT_HEADER);
            tableCell.AddInnerHTMLString("**Page Name**");
            tableRow.AddInnerHTMLElement(tableCell);
            
            result.AddInnerHTMLElement(tableRow);
            
            // output the results
            for (int i = 0; i new HTMLTableRow();
                
                // row coloring
                if ((0 == i) || (((i + 1) / 2) == (i / 2))) {
                    // it's even
                    tableRow.SetStyleClass(PTStyleClass.LIST_ITEM_TWO_BG);
                } else {
                    // it's odd
                    tableRow.SetStyleClass(PTStyleClass.LIST_ITEM_ONE_BG);
                }
    
                tableCell = new HTMLTableCell();
                tableCell.SetVAlign(CommonHTMLStrings.MIDDLE);
                tableCell.AddInnerHTMLString(dbrow.GetString("GADGETNAME"));
                tableRow.AddInnerHTMLElement(tableCell);
    
                tableCell = new HTMLTableCell();
                tableCell.SetVAlign(CommonHTMLStrings.MIDDLE);
                tableCell.AddInnerHTMLString(dbrow.GetString("COMMUNITYNAME"));
                tableRow.AddInnerHTMLElement(tableCell);
                
                tableCell = new HTMLTableCell();
                tableCell.SetVAlign(CommonHTMLStrings.MIDDLE);
                tableCell.AddInnerHTMLString(dbrow.GetString("PAGENAME"));
                tableRow.AddInnerHTMLElement(tableCell);
                
                result.AddInnerHTMLElement(tableRow);
            }
    
            return result;
        }
    
        private int getPortletId() {
            try {
                return Integer.parseInt(GetTagAttributeAsString(PORTLETID));
            } catch (Exception ex) {
                return -1;
            }
        }
    
        private int getMaxRows() {
            try {
                return Integer.parseInt(GetTagAttributeAsString(MAXROWS));
            } catch (Exception ex) {
                return 0;
            }
        }
        
        private IPTSession getSession() {
            return (IPTSession) GetEnvironment().GetUserSession();
        }
    
        private boolean hasAdminAccess() {
            return TaskAPIUIUser.HasAdminLinkAccess((AActivitySpace) this
                    .GetEnvironment());
        }
    }

### Caveat Emptor

As [Uncle Ben][3] would say, "With great power comes great responsibility." The power I gave you above may also allow you to perform INSERT's, UPDATE's and DELETE's, which I strongly caution against. Not only that, but the InternalSession object doesn't perform all those nifty security checks that happen when we use a normal session (notice the hasAdminAccess function), so make sure you either do your own authentication, or limit the amount of information you provide.

Happy querying.

 [1]: http://edocs.bea.com/alui/devdoc/docs60/References/Portal_API_Documentation.htm
 [2]: http://en.wikipedia.org/wiki/Big_O_notation
 [3]: http://en.wikipedia.org/wiki/Uncle_Ben  