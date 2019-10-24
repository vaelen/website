---
date: "2010-05-20T01:02:33"
lastmod: "2010-05-20T01:02:33"
title: "XML Generation in RPG"
---
The company I work for makes heavy use of their IBM Power i midrange servers (previously known as AS/400 or iSeries servers).  A lot of their software is written in the [RPG programming language](http://en.wikipedia.org/wiki/IBM_RPG), which IBM originally developed back in the 1960s.  The language was originally written to generate reports and lacked many "modern" programming features, such as IF statements and subroutines, which were added in RPG III.

Since starting at my current company, I've been trying to learn the current version of RPG, which is RPG IV (aka RPGLE or ILE/RPG).  Most of the running code that I see in RPG is actually written using RPG III syntax despite the fact that RPG IV has been out since 1994.  This is mostly due to the fact that much of it was either generated programmatically or was written before 1994.  My goal in learning RPG isn't to become proficient enough to program RPG for a living, but instead to become proficient enough to help our organization transition their existing systems to more modern technologies as needed.  However, my "outsider" view of RPG (coming from a Java/Perl/Ruby/etc background) has helped me do some things with it that long time RPG programmers might not think of trying to do.  This is an example of that.

Like most large enterprise development shops these days, we use a Service Oriented Architecture (SoA) for most of our newer Java and .NET services.  The problem is integrating our legacy RPG systems into these newer SoA systems.  The cleanest approach is to use data queues (aka JMS queues).  RPG can use these queues natively without having to make use of outside libraries as it would need to do in order to make socket calls.  The problem is how to structure the data that is placed on the queue.  The best thing to do would be to place a SOAP message - a type of XML document - onto the queue.  This way the SoA systems could easily pick the message up off the queue and process it just like any other SOAP message they receive.  However, this turns out to be problematic because it is difficult to produce properly formatted XML from RPG without the use of external libraries.

The good news is that there are products available that can help solve this problem.  (For example, [The RPG XML Suite](http://www.rpg-xml.com/about.aspx) and [Scott Klement's presentation](http://www.scottklement.com/presentations/#FREEXML) on using free software to produce XML from RPG.)  However, for various reasons we are unable to use these solutions, and since we have a large pool of RPG programmers who don't yet know Java or .NET, there will still be a need to develop applications in RPG and have them interact via XML with our SoA applications.

To help solve this problem, and to help me learn RPG, I wrote a program that lets the programmer generate a sort of "mini DOM" for an XML document that can then produce a properly formatted and escaped XML string.  Below is the code for the program.  However, please note that I haven't tested it throughly and since I am a novice RPG programmer it is bound to have bugs in it.  I also used lots of "newer" RPG programming concepts, such as pointers, dynamic memory allocation, variable length null terminated strings, and subprocedures (which are like subroutines except that they have their own local variables and call stack that allow them to be run recursively.)

``` rpg
      // This program implements a simple XML DOM Builder.
      // Written by Andrew Young <andrew@vaelen.org>

     D MAX_NODES       C                   65535

     D NODE_DEF        DS
     D   nodeId                       5U 0 INZ(1)
     D   nodeParent                   5U 0 INZ(0)
     D   nodeType                     1A   INZ('E')
     D   nodeName@                     *   INZ(*NULL)
     D   nodeNameLen                  5U 0 INZ(0)
     D   nodeValue@                    *   INZ(*NULL)
     D   nodeValLen                   5U 0 INZ(0)

     D NODE_LIST       DS
     D nodes                           *   DIM(MAX_NODES)
     D nodeEnd                        5U 0 INZ(1)

     D buildXML        PR         65535A   VARYING
     D                                 *   VALUE

     D createNode      PR              *
     D                                5U 0 VALUE

     D setNodeName     PR             3U 0
     D                                 *   VALUE
     D                            65535A   VALUE VARYING

     D getNodeName     PR         65535A   VARYING
     D                                 *   VALUE

     D setNodeValue    PR             3U 0
     D                                 *   VALUE
     D                            65535A   VALUE VARYING

     D getNodeValue    PR         65535A   VARYING
     D                                 *   VALUE

     D getNode         PR              *
     D                                5U 0 VALUE

     D getParentNode   PR              *
     D                                 *   VALUE

     D nodeTest        PR             3U 0

     D printNode       PR             3U 0
     D                                 *   VALUE

     D displayString   PR             3U 0
     D                            65535A   VARYING VALUE

     D escapeXML       PR         65535A   VARYING
     D                            65535A   VARYING VALUE

     D cleanup         PR

      /FREE
       nodeTest();
       cleanup();
       EXSR doPause;
       *INLR = *ON;
       RETURN;
      /END-FREE

      * This method pauses the output
     C     doPause       BEGSR
     C                   DSPLY                   PAUSE             1
     C                   ENDSR

     P cleanup         B
     D i               S             10I 0 INZ(1)
     D ret             S              3U 0
     D node@           S               *
     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
      /FREE
       DOW i < nodeEnd;
         node@ = getNode(i);
         IF node@ <> *NULL;
           IF node.nodeName@ <> *NULL;
             DEALLOC node.nodeName@;
             node.nodeName@ = *NULL;
           ENDIF;
           IF node.nodeValue@ <> *NULL;
             DEALLOC node.nodeValue@;
             node.nodeValue@ = *NULL;
           ENDIF;
           DEALLOC node@;
           node@ = *NULL;
         ENDIF;
         i = i + 1;
       ENDDO;
      /END-FREE
     P cleanup         E

     P nodeTest        B
     D                 PI             3U 0

     D node@           S               *
     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D nodeIndex       S              5U 0
     D ret             S              3U 0

      /FREE
       node@ = createNode(0);
       ret = setNodeName(node@:'documents');

       node@ = createNode(node.nodeId);
       ret = setNodeName(node@:'document');

       node@ = createNode(node.nodeId);
       ret = setNodeName(node@:'name');
       node.nodeType = 'A';
       ret = setNodeValue(node@:'My First Document');

       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       node.nodeType = 'T';
       ret = setNodeValue(node@:'Some text.');

       node@ = getParentNode(node@);
       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       ret = setNodeName(node@:'document');

       node@ = createNode(node.nodeId);
       node.nodeType = 'A';
       ret = setNodeName(node@:'name');
       ret = setNodeValue(node@:'A Second Document <>&"&"><');

       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       ret = setNodeName(node@:'field');

       node@ = createNode(node.nodeId);
       node.nodeType = 'A';
       ret = setNodeName(node@:'name');
       ret = setNodeValue(node@:'url');

       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       node.nodeType = 'T';
       ret = setNodeValue(node@:'http://www.google.com/');

       node@ = getParentNode(node@);
       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       ret = setNodeName(node@:'description');

       node@ = createNode(node.nodeId);
       node.nodeType = 'T';
       ret = setNodeValue(node@:'Here is a link: ');

       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       setNodeName(node@:'a');

       node@ = createNode(node.nodeId);
       node.nodeType = 'A';
       setNodeName(node@:'href');
       setNodeValue(node@:'http://www.google.com/');

       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       node.nodeType = 'T';
       setNodeValue(node@:'Google');

       node@ = getParentNode(node@);

       node@ = getParentNode(node@);

       node@ = createNode(node.nodeId);
       node.nodeType = 'T';
       setNodeValue(node@:', and here is some text that needs to be escaped: '+
           '"<>"&><&');

       nodeIndex = 1;
       DOW nodeIndex < nodeEnd;
         node@ = getNode(nodeIndex);
         ret = printNode(node@);
         nodeIndex = nodeIndex + 1;
       ENDDO;

       ret = displayString(buildXML(getNode(1)));

       RETURN 0;
      /END-FREE
     P nodeTest        E

       // This routine prints out the value of the current node
     P printNode       B
     D                 PI             3U 0
     D node@                           *   VALUE

     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D i               S              3U 0
      /FREE
       i = displayString('Node: ' + %char(node.nodeId));
       i = displayString('  Parent: ' + %char(node.nodeParent));
       i = displayString('    Type: ' + node.nodeType);
       i = displayString('    Name: ' + getNodeName(node@));
       i = displayString('   Value: ' + getNodeValue(node@));
       RETURN 0;
      /END-FREE
     P printNode       E

       // This routine write the output to standard out using DSPLY.
     P displayString   B
     D                 PI             3U 0
     D output                     65535A   VARYING VALUE

     D message         S             52A   VARYING
     D i               S             10I 0 INZ(0)
     D j               S             10I 0 INZ(0)
      /FREE
       i = 1;
       j = %len(%trim(output));
       DOW i < j;
         message = %subst(output:i);
         DSPLY message;
         i = i + 52;
       ENDDO;
       RETURN 0;
      /END-FREE
     P displayString   E

     P getNode         B
     D                 PI              *
     D nodeIndex                      5U 0 VALUE
      /FREE
       RETURN nodes(nodeIndex);
      /END-FREE
     P getNode         E

     P getParentNode   B
     D                 PI              *
     D node@                           *   VALUE

     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D parentNode@     S               *
      /FREE
       IF node.nodeParent > 0;
         parentNode@ = getNode(node.nodeParent);
       ENDIF;
       RETURN parentNode@;
      /END-FREE
     P getParentNode   E

     P createNode      B
     D                 PI              *
     D parentIndex                    5U 0 VALUE

     D nodeIndex       S              5U 0
     D node@           S               *
     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D i               S              3U 0
      /FREE
       node@ = %alloc(%size(NODE_DEF));
       node.nodeId = nodeEnd;
       node.nodeType = 'E';
       node.nodeParent = parentIndex;
       nodes(nodeEnd) = node@;
       nodeEnd = nodeEnd + 1;
       RETURN node@;
      /END-FREE
     P createNode      E

     P setNodeName     B
     D                 PI             3U 0
     D node@                           *   VALUE
     D value                      65535A   VALUE VARYING

     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D i               S              3U 0
      /FREE
       IF node.nodeName@ <> *NULL;
         // Deallocate currently used space
         DEALLOC node.nodeName@;
         node.nodeName@ = *NULL;
       ENDIF;
       // Allocate new space
       node.nodeNameLen = %len(value);
       node.nodeName@ = %alloc(node.nodeNameLen+1);
       %str(node.nodeName@:node.nodeNameLen+1) =
           %subst(value:1:node.nodeNameLen);
       RETURN 0;
      /END-FREE
     P setNodeName     E

     P getNodeName     B
     D                 PI         65535A   VARYING
     D node@                           *   VALUE

     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D value           S          65535A   VARYING
     D i               S              3U 0
      /FREE
       IF node.nodeName@ <> *NULL;
         value = %str(node.nodeName@:node.nodeNameLen+1);
       ENDIF;
       RETURN value;
      /END-FREE
     P getNodeName     E

     P setNodeValue    B
     D                 PI             3U 0
     D node@                           *   VALUE
     D value                      65535A   VALUE VARYING

     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D i               S              3U 0
      /FREE
       IF node.nodeValue@ <> *NULL;
         // Deallocate currently used space
         DEALLOC node.nodeValue@;
         node.nodeValue@ = *NULL;
       ENDIF;
       // Allocate new space
       node.nodeValLen = %len(value);
       node.nodeValue@ = %alloc(node.nodeValLen+1);
       %str(node.nodeValue@:node.nodeValLen+1) =
           %subst(value:1:node.nodeValLen);
       RETURN 0;
      /END-FREE
     P setNodeValue    E

     P getNodeValue    B
     D                 PI         65535A   VARYING
     D node@                           *   VALUE

     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D value           S          65535A   VARYING
      /FREE
       IF node.nodeValue@ <> *NULL;
         value = %str(node.nodeValue@:node.nodeValLen+1);
       ENDIF;
       RETURN value;
      /END-FREE
     P getNodeValue    E

     P buildXML        B
     D                 PI         65535A   VARYING
     D node@                           *   VALUE

     D childNode@      S               *
     D outp            S          65535A   VARYING
     D i               S             10I 0
     D node            DS                  LIKEDS(NODE_DEF) BASED(node@)
     D childNode       DS                  LIKEDS(NODE_DEF) BASED(childNode@)

      /FREE
       // This function writes out the XML structure for the current node and
       //     its children.
       IF node.nodeType = 'E';
         // Write start of element
         outp = outp + '<' + %trim(getNodeName(node@));
         // Look for child attributes
         i = 1;
         DOW i < nodeEnd;
           childNode@ = getNode(i);
           IF childNode.nodeType = 'A' AND childNode.nodeParent = node.nodeId;
             outp = outp + ' ' + %trim(getNodeName(childNode@)) + '="' +
                 escapeXML(getNodeValue(childNode@)) + '"';
           ENDIF;
           i = i + 1;
         ENDDO;
         outp = outp + '>';
         // Look for child elements and text nodes
         i = 1;
         DOW i < nodeEnd;
           childNode@ = getNode(i);
           IF childNode.nodeParent = node.nodeId;
             IF childNode.nodeType = 'T';
               outp = outp + escapeXML(getNodeValue(childNode@));
             ELSEIF childNode.nodeType = 'E';
               outp = outp + buildXML(childNode@);
             ENDIF;
           ENDIF;
           i = i + 1;
         ENDDO;
         outp = outp + '</' + %trim(getNodeName(node@)) + '>';
       ENDIF;  // node.nodeType = 'E'

       RETURN outp;

      /END-FREE

     P buildXML        E

     P escapeXML       B
     D                 PI         65535A   VARYING
     D input                      65535A   VARYING VALUE

     D inputSize       S              5U 0 INZ(0)
     D i               S              5U 0 INZ(1)
     D currentChar     S              1A   INZ(*BLANKS)
     D outputChar      S              6A   VARYING
     D output          S          65535A   VARYING
      /FREE
       inputSize = %len(input);
       DOW i <= inputSize;
         currentChar = %subst(input:i:1);
         outputChar = currentChar;
         IF currentChar = '&';
           outputChar = '&amp;amp;';
         ELSEIF currentChar = '<';
           outputChar = '&amp;lt;';
         ELSEIF currentChar = '>';
           outputChar = '&amp;gt;';
         ELSEIF currentChar = '"';
           outputChar = '&amp;quot;';
         ENDIF;
         output = output + outputChar;
         i = i + 1;
       ENDDO;
       RETURN output;
      /END-FREE
     P escapeXML       E
```
