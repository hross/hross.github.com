---
layout: post
title: Update Publisher Publishing Targets
categories: [webcenter]
---

I have recently been working on a utility for porting ALUI databases from a production environment to a development environment. [Fabien Sanglier][1] started this effort, and I hope to have some code to contribute to his [ALUI toolbox project][2] very soon.

In the meantime, however, I have been banging my head against the pain that is migrating Publish and Preview target URL's in Publisher. These URL's are stored in a binary BLOB in the Publisher database, and are actually serialized Java classes, making them extremely difficult to update (especially when you don't have access to the original Publisher source code).

My original plan was to wrap all of this stuff into one “uber-utility” and then blog about it. Recently, though, I saw this post on the Oracle Webcenter Interaction discussion forums: [http://forums.oracle.com/forums/thread.jspa?threadID=900736&amp;tstart=0][3] and it made me think I should probably post the code for migrating Publishing Targets, for the benefit of the sanity of the community at large.

Here is a [link to a jar file which will update Publisher publish targets][4]. If you crack the jar file with a zip editor, you will be able to update the *configuration.properties* file in the root directory to suit your needs.

I took the liberty of including the Publisher classes in my own jar, making it simpler to run from a command line. To run it, you will only need to download the correct jdbc driver for your database:

[Oracle JDBC Driver][5]

[SQL Server JDBC Driver][6]

Next, simply execute it from a java command line with the driver in your classpath, like so:

    java -cp updatepublishtargets.jar;ojdbc14.jar net.hross.content.UpdatePublishTargets

Note that the utility is in debug mode by default, so nothing will happen to your Publisher database until you set debug to false in the configuration, although now is probably a good time to let you know that ***I provide no warranties of any kind with this code***.

In order to build and run the source, you will need the *content.jar* and *dom4j.jar* found in the WEB-INF/lib directory of your *ptcs.war*. Here is the relevant source code, in case you are looking to build your own version of the utility (source is also in the jar):

    package net.hross.content;
    
    import java.io.ByteArrayInputStream;
    import java.io.ByteArrayOutputStream;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.ObjectInputStream;
    import java.io.ObjectOutputStream;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.sql.Statement;
    import java.util.HashMap;
    import java.util.Iterator;
    import java.util.Map;
    
    import net.hross.utility.Configuration;
    
    import com.plumtree.content.data.AttributeKey;
    import com.plumtree.content.data.impl.RdbiPublishingTarget;
    
    public class UpdatePublishTargets {
    
        public static void main(String[] args) {
            Connection connection = Configuration.getConnection();
    
            if (null == connection) {
                System.out.println("Unable to connect to database. Exiting.");
            }
    
            int directoryId = Integer.parseInt(Configuration
                    .getString(Configuration.CONFIG_DIRECTORY_ID));
            boolean debug = Boolean.parseBoolean(Configuration
                    .getString(Configuration.CONFIG_DEBUG_MODE));
            String newPublishTarget = Configuration
                    .getString(Configuration.CONFIG_PUBLISH_TARGET);
    
            System.out.println("Updating publish targets for directory ID: "
                    + directoryId);
    
            System.out.println();
            if (debug) {
                System.out.println("** DEBUG MODE ON ** Nothing will be updated.");
            } else {
                System.out.println("** DEBUG MODE OFF ** This is happening for real.");
            }
            System.out.println();
    
            updatePublishTarget(connection, directoryId, newPublishTarget, debug);
        }
    
        /***
         * Update the publishing target for a specified directory ID (-1 for all
         * items).
         * 
         * @param connection
         *            Publisher database connection.
         * @param directoryId
         *            Directory ID to update. -1 for all items.
         * @param newPublishTarget
         *            - New publishing target.
         * @param debug
         *            - if true, no replace will be made, data will just be output.
         */
        public static void updatePublishTarget(Connection connection,
                int directoryId, String newPublishTarget, boolean debug) {
    
            // create a statement to query the directory id
            try {
    
                // create prepared statement for directory query
                PreparedStatement psDirectory = null;
                if (directoryId &gt; 0) {
                    psDirectory = connection
                            .prepareStatement("SELECT * FROM PCSDIRECTORY WHERE ITEMTYPE=0 AND DIRECTORYID=?");
                    psDirectory.setInt(1, directoryId);
                } else {
                    psDirectory = connection
                            .prepareStatement("SELECT * FROM PCSDIRECTORY WHERE ITEMTYPE=0");
                }
                ResultSet rs = psDirectory.executeQuery();
    
                // loop through any rows we need to check
                while (rs.next()) {
    
                    // get basic info about the object
                    String itemName = rs.getString("ITEMNAME");
                    int size = rs.getInt("DATASIZE");
                    
                    // reset directory ID in case it was generic
                    directoryId = rs.getInt("DIRECTORYID");
    
                    // get binary input stream
                    InputStream input = rs.getBinaryStream("DATABYTES");
    
                    // if there's actually some settings, let's check them
                    if ((null != input) &amp;&amp; (0 != size)) {
    
                        // generic catch statement for problems with this item
                        try {
                            byte[] buffer = new byte[size];
                            input.read(buffer);
    
                            // load the hash map from the database
                            Map map = (HashMap) deserialize(buffer);
    
                            // loop through the keys in the hash map
                            Iterator keys = map.keySet().iterator();
                            while (keys.hasNext()) {
                                Object key = keys.next();
    
                                // this should probably always be true
                                if (key.getClass().equals(AttributeKey.class)) {
                                    AttributeKey akey = (AttributeKey) key;
    
                                    // if we found a publishing target...
                                    if (akey.getKeyString().equals(
                                            "PUBLISHING_TARGET")) {
                                        System.out.println();
                                        System.out.println("--------------------");
                                        System.out
                                                .println("Updating publishing target for:");
                                        System.out.println(directoryId + " - "
                                                + itemName);
    
                                        // get the publishing target info
                                        RdbiPublishingTarget val = (RdbiPublishingTarget) map
                                                .get(key);
                                        String publishTarget = val
                                                .getPublishDetail()
                                                .getTargetLocation();
                                        String publishBrowser = val
                                                .getPublishDetail()
                                                .getBrowserLocation();
                                        String previewTarget = val
                                                .getPreviewDetail()
                                                .getTargetLocation();
                                        String previewBrowser = val
                                                .getPreviewDetail()
                                                .getBrowserLocation();
                                        String ftpUser = val.getPublishDetail()
                                                .getUsername();
                                        String ftpPassword = val.getPublishDetail()
                                                .getPassword();
    
                                        System.out
                                                .println("Publish  browser location: "
                                                        + publishBrowser);
                                        System.out.println("Preview target: "
                                                + previewTarget);
                                        System.out
                                                .println("Preview browser location: "
                                                        + previewBrowser);
                                        System.out.println("FTP user: " + ftpUser);
                                        System.out.println("FTP password: "
                                                + ftpPassword);
                                        System.out.println("Old publish target: "
                                                + publishTarget);
                                        System.out.println("New publish target: "
                                                + newPublishTarget);
    
                                        // if we are doing this for real, update
                                        // values
                                        if (!debug) {
                                            val.setTargetValues(newPublishTarget,
                                                    publishBrowser, previewTarget,
                                                    previewBrowser, ftpUser,
                                                    ftpPassword);
    
                                            map.put(key, val);
    
                                            // update the directory
                                            serializeToDirectory(connection,
                                                    directoryId, map);
                                            System.out.println("Update successful.");
                                        }
                                        System.out.println("--------------------");
                                        System.out.println();
                                    }
                                }
                            }
    
                            // clean up
                            input.close();
                        } catch (IOException ex) {
                            System.out.println("Something bad happened.");
                            ex.printStackTrace();
                        }
                    } // if null
                } // while next rs
            } catch (SQLException ex) {
                System.out.println("Something bad happened.");
                ex.printStackTrace();
            }
    
            System.out.println("Procedure successfully completed.");
        }
    
        private static Object deserialize(byte bytes[]) {
            try {
                ByteArrayInputStream byteStream = new ByteArrayInputStream(bytes);
                ObjectInputStream objectStream = new ObjectInputStream(byteStream);
                return objectStream.readObject();
            } catch (Exception ex) {
                return null;
            }
        }
    
        private static void serializeToDirectory(Connection conn, int directoryId,
                Object obj) throws IOException, SQLException {
            byte bytes[] = getBytes(obj);
            ByteArrayInputStream byteStream = new ByteArrayInputStream(bytes);
    
            PreparedStatement ps = conn
                    .prepareStatement("UPDATE PCSDIRECTORY SET DATASIZE=?, DATABYTES=? WHERE DIRECTORYID=?");
            ps.setInt(1, bytes.length);
            ps.setBinaryStream(2, byteStream, bytes.length);
            ps.setInt(3, directoryId);
            ps.execute();
            conn.commit();
        }
    
        public static byte[] getBytes(Object obj) throws java.io.IOException {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(obj);
            oos.flush();
            oos.close();
            bos.close();
            byte[] data = bos.toByteArray();
            return data;
        }
    }

 [1]: http://fsanglier.blogspot.com/
 [2]: http://code.google.com/p/alui-toolbox/
 [3]: http://forums.oracle.com/forums/thread.jspa?threadID=900736&amp;tstart=0 "http://forums.oracle.com/forums/thread.jspa?threadID=900736&amp;tstart=0"
 [4]: /assets/updatepublishtargets.jar
 [5]: http://www.oracle.com/technology/software/tech/java/sqlj_jdbc/index.html
 [6]: http://msdn.microsoft.com/en-us/data/aa937724.aspx  