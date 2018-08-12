<html>
 <body>
  <div class="content"> 
   <h3>Protocol Buffers（<a href="https://www.charlesproxy.com/documentation/using-charles/protocol-buffers/">原文链接</a>）</h3> 
   <h4>背景</h4> 
   <p>引自<a href="https://developers.google.com/protocol-buffers/">Protocol Buffers官网</a>：</p> 
   <blockquote> 
    <p>Protocol buffers是一种由谷歌提供的，用于序列化和反序列化数据的解决方案。它的特点是语言中立，平台中立，且对比于XML，体积更小，速度更快，使用更简单。你只需将你的数据结构声明一次，就可以使用生成的特定源码，来读写你来自各种数据流和语言的结构化数据。</p> 
   </blockquote> 
   <p>Protocol Buffers的序列化格式是一个二进制编码的格式，并不容易被人类阅读。鉴于Protocol Buffers的信息通常都是获取自HTTP，我们添加了完整的便于人阅读和编辑的Protocol Buffers编辑工具。</p> 
   <p>Charles当前支持2.4.1版本的Protocol Buffers，并且可以支持大量更早版本的Protocol Buffers。</p> 
   <h4>概述</h4> 
   <p>Charles会检查，当一个HTTP请求或响应的header的<code>Content-Type</code>中有<code>application/x-protobuf</code>或<code>application/x-google-protobuf</code>的MIME类型时，这个请求就含有Protocol Buffers的信息。这时你就可以看到两个新的内容视图<a href="#text_viewer">Protobuf Text Viewer</a>及<a href="#structured_viewer">Protobuf Structured Viewer</a>。
   </p> 
   <p>In order for these viewers to be able to display the message content they need access to the protocol buffers descriptor for the message(s) that are contained in the HTTP body content. Charles looks for&nbsp;<code>desc</code>&nbsp;and&nbsp;<code>messageType</code>&nbsp;parameters in the content-type to discover the location of the&nbsp;<em>FileDescriptorSet</em>&nbsp;(<code>*.desc</code>&nbsp;file) and fully qualified message type name, it then uses these to retrieve and load the appropriate descriptor for the message(s). A protocol buffers FileDescriptorSet can be generated from a&nbsp;<code>*.proto</code>&nbsp;file by the protocol buffers compiler (protoc) by using the&nbsp;<code>-o</code>&nbsp;or&nbsp;<code>--descriptor_set_out</code>&nbsp;option e.g.&nbsp;<code>protoc -oModel.desc Model.proto</code>.</p> 
   <p>Finally the HTTP body content may contain a single message or a list of messages which have been serialised using the standard protocol buffers length delimited format. To determine whether the HTTP body contains a single message or a delimited list of messages an optional<code>delimited</code>&nbsp;parameter in the content-type is looked for. This must be present and have the value&nbsp;<code>true</code>&nbsp;to indicate a delimited list of messages has been sent. When a delimited list of messages has been sent all messages must be of the same message type.</p> 
   <p>This means a complete&nbsp;<code>Content-Type</code>&nbsp;header will look like:<code>Content-Type: application/x-protobuf; desc=&quot;http://localhost/Model.desc&quot;; messageType=&quot;com.xk72.sample.PurchaseOrder&quot;; delimited=true</code></p> 
   <h4>Loading the FileDescriptorSet</h4> 
   <p>Charles expects the&nbsp;<code>desc</code>&nbsp;parameter of the&nbsp;<code>Content-Type</code>&nbsp;header to contain a valid URL pointing to the location of the FileDescriptorSet for the protocol buffer message. This can be any valid URL but is usually a&nbsp;<code>file</code>&nbsp;or&nbsp;<code>http</code>&nbsp;URL. Charles attempts to retrieve the FileDescriptorSet every time one one of the Protobuf viewers is selected to display the HTTP body content, this means that the&nbsp;<code>desc</code>&nbsp;URLs are not resolved until needed which is critical when large numbers of protocol buffer transactions are being recorded by Charles.</p> 
   <h4>FileDescriptorSet Caching<a id="caching" name="caching"></a></h4> 
   <p>Typically you still get many transactions using the same protocol buffer message types, so Charles does implement caching of the FileDescriptorSets based upon&nbsp;<a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html">standard HTTP 1.1 caching</a>&nbsp;for&nbsp;<code>http</code>&nbsp;URLs and the last modified timestamp for&nbsp;<code>file</code>&nbsp;URLs.</p> 
   <p>This means you can ensure the performance of the FileDescriptorSet loading by ensuring the web server which is serving up the files is setting appropriate HTTP 1.1 caching and/or validation (Last-Modified and ETag) headers. Caching can also be effectively disabled by setting the appropriate HTTP 1.1 no-cache headers.</p> 
   <p>When no HTTP caching or validation headers are present on the FileDescriptorSet responses then Charles calculates its own heuristic expiration time for cached FileDescriptorSets. The length of the time Charles will cache these resources for is configurable in the&nbsp;<em>Preferences-&gt;Viewers</em>dialog in Charles, it defaults to 5 minutes. This can be set to 0 to disable the heuristic caching.</p> 
   <p><em>Please note this caching is implemented for the resolving of the&nbsp;<code>desc</code>&nbsp;URLs only, it does not in any way change the behaviour of Charles with respect to the HTTP transactions being recorded.</em></p> 
   <h4 id="text_viewer">Protobuf Text Viewer</h4> 
   <p>The Protobuf Text body content viewer displays the default textual representation of the protocol buffer message. This is the same format implemented by the&nbsp;<code>com.google.protobuf.TextFormat</code>&nbsp;class in the Java API, the&nbsp;<code>google.protobuf.text_format.h</code>&nbsp;C++ API or the<code>google.protobuf.text_format</code>&nbsp;Python module that come with the Protocol Buffers distribution.</p> 
   <p>When displaying a delimited list of messages each message is separated by the textual delimiter:<code>&gt;--------------------------------next message--------------------------------&lt;</code></p> 
   <h4 id="structured_viewer">Protobuf Viewer</h4> 
   <h5>Basic Message Display</h5> 
   <p>The Protobuf body content viewer shows a tree-structured view of the protocol buffer message, showing the full hierarchical structure of the message with all fields and sub-messages.</p> 
   <p>In the structured viewer the first column displays the defined field name for each field in the tree-structure defined by the Protocol Buffer message definition.</p> 
   <p>The second column displays the type of the field:</p> 
   <ul> 
    <li>Scalar fields are given their&nbsp;<code>.proto</code>&nbsp;type name e.g.&nbsp;<code>string</code>,&nbsp;<code>uint32</code>,&nbsp;<code>bool</code>,&nbsp;<code>sfixed64</code>&nbsp;etc</li> 
    <li>Enumerations are labeled with the fully qualified type name of the enumeration prefixed with&nbsp;<code>Enum</code></li> 
    <li>Message typed fields are given the fully qualified type name of the message</li> 
    <li>Any repeating field is prefixed with&nbsp;<code>Repeating</code>&nbsp;e.g.&nbsp;<code>Repeating string</code></li> 
   </ul> 
   <p>The third column displays information about the value of the field:</p> 
   <ul> 
    <li>Scalar fields have a textual representation of their value displayed</li> 
    <li>Enumerations have the defined name for their value displayed</li> 
    <li>Repeating fields display the number of repeats that exist for the field</li> 
   </ul> 
   <p>Message typed and repeating fields can be expanded to display the individual fields or repeats they encapsulate. Note that for&nbsp;<code>string</code>&nbsp;fields what is being displayed is not the literal string value but an encoding of it to support representing non-printable characters. The encoding used is the format used for string literals in C code, so that newlines become&nbsp;<code>\n</code>, tabs become&nbsp;<code>\t</code>&nbsp;and so on.</p> 
   <p>Any field may be double-clicked to open a pop-up dialog containing a string representation of the value of that field. This is useful when the field content is too large to be easily displayed in the table. Also when a message typed field is double-clicked the entire content of the message contained in that field including all its child fields is displayed.</p> 
   <h5>Handling of Missing Optional Fields</h5> 
   <p>With protocol buffer messages fields that are defined in the message definition as optional do not have to be included in the message. In this situation where the field does not explicitly exist in the message it still has an implied value. This will be the default value if one is defined for the field in the message definition or else it gets the normal default value for its type e.g.&nbsp;<code>0</code>&nbsp;for the number types,&nbsp;<code>false</code>&nbsp;for&nbsp;<code>bool</code>&nbsp;types and the empty string for&nbsp;<code>string</code>&nbsp;types.</p> 
   <p>Usually these unspecified fields are hidden as it is presumed they are of little interest. This includes hiding repeating fields that have 0 repeats. You can configure the viewer to display these unspecified fields in the&nbsp;<em>Preferences-&gt;Viewers</em>&nbsp;dialog in Charles.</p> 
   <p>When unspecified fields are being displayed their field name, type information and implied or default value show up in&nbsp;<em>italics</em>&nbsp;in the viewer.</p> 
   <h5>Unknown Fields</h5> 
   <p>With protocol buffers messages it is possible to get unknown fields in the message content, this typically happens when the message definition has changed and an older message that contains fields that no longer exist in the message definition is being parsed.</p> 
   <p>When this happens the structured viewer displays an&nbsp;<code>Unknown Fields</code>&nbsp;child node in the message hierarchy which can be expanded to see the available information pertaining to the unknown fields. Because the meta-data that is normally found for these fields in the message definition is not available all you can see is the field's numeric tag, serialised wire type and serialised value. The possible wire types are:</p> 
   <ul> 
    <li>Varint</li> 
    <li>32-bit</li> 
    <li>64-bit</li> 
    <li>Length-delimited</li> 
   </ul> 
   <p>see the&nbsp;<a href="https://developers.google.com/protocol-buffers/docs/encoding#structure">protocol buffers encoding documentation</a>&nbsp;for information on the mapping from&nbsp;<code>.proto</code>&nbsp;scalar types to these wire types and how to decode the serialised values.</p> 
   <h4>Unknown Messages</h4> 
   <p>There are some situations where the protocol buffers descriptor for the message is not available. For example this could happen when the<code>desc</code>&nbsp;or&nbsp;<code>messageType</code>&nbsp;parameters were not specified in the&nbsp;<code>Content-Type</code>&nbsp;header or there was an error resolving the URL provided in the<code>desc</code>&nbsp;parameter.</p> 
   <p>When this happens the protocol buffers message is parsed as an&nbsp;<em>Unknown</em>&nbsp;message, in effect this means that all the fields in the message become unknown fields and are displayed with just the minimal information we can discover about them from the wire format. The error that caused the message to be parsed as an Unknown message is also displayed in the viewer, hopefully allowing you to fix whatever caused the problem.</p> 
   <h4>Configuration Options</h4> 
   <p>There are some aspects of the Protocol Buffers functionality that is user configurable. These options are available in the&nbsp;<em>Viewers</em>&nbsp;section of the<em>Preferences</em>&nbsp;dialog. The&nbsp;<em>Preferences</em>&nbsp;dialog is activated from the Edit menu (or Charles menu on Mac OS X).</p> 
   <ul> 
    <li><strong>Hide Unspecified Fields</strong>&nbsp;- when selected optional fields that have not been specified in a protocol buffer message are hidden when that message is displayed in the structured viewer, when not selected the unspecified fields are displayed in italics along with the default or implied value that they take</li> 
    <li><strong>Cache Protobuf Descriptors</strong>&nbsp;- enables the caching of protocol buffers descriptors as&nbsp;<a href="#caching">described in the caching section</a>, when not selected descriptors will not be cached</li> 
    <li><strong>Heuristic Cache TTL</strong>&nbsp;- when caching is enabled this is the cache TTL or expiry time used for descriptors retrieved from an HTTP URL where no HTTP caching headers are specified, set to 0 to never cache when there are no explicit HTTP caching headers</li> 
    <li><strong>Clear Cached Resources</strong>&nbsp;- clicking this button immediately clears all cached protocol buffers descriptors</li> 
   </ul> 
   <h4>Comparing messages</h4> 
   <p>When you have exactly two transactions selected in either the structure or sequence views you can compare their content by using the&nbsp;<em>Compare</em>command from the right-click context menu.</p> 
   <p>When both of the transactions contain protocol buffer messages as well as the standard comparison viewers normally available you have the option of using the&nbsp;<a href="#text_viewer">Protobuf Text Viewer</a>&nbsp;or&nbsp;<a href="#structured_viewer">Protobuf Structured Viewer</a>&nbsp;to view the comparison between the transactions.</p> 
   <h4>Editing messages</h4> 
   <p>The&nbsp;<a href="#text_viewer">Protobuf Text Viewer</a>&nbsp;and&nbsp;<a href="#structured_viewer">Protobuf Structured Viewer</a>&nbsp;may also be used when editing the content of an HTTP request. When you choose to edit an HTTP request if it is identified as a protocol buffers request (by having a content type of&nbsp;<code>application/x-protobuf</code>&nbsp;or<code>application/x-google-protobuf</code>) then the two protobuf specific viewers will be available as options for editing the request content.</p> 
   <h5>Protobuf Text Viewer - Edit mode</h5> 
   <p>The Protobuf Text viewer in edit mode simply allows you to edit the human readable textual representation of the protocol buffers message. This text is both generated and then reparsed (after editing) into a serialised protocol buffers message using the standard Protocol Buffers library API, that is the&nbsp;<code>com.google.protobuf.TextFormat</code>&nbsp;class in the Java API, the&nbsp;<code>google.protobuf.text_format.h</code>&nbsp;C++ API or the<code>google.protobuf.text_format</code>&nbsp;Python module.</p> 
   <p>One quirk of the implementation you may need to be aware of is that if the original message contained unknown fields these are represented in the text generated by the protocol buffers text formatter but they cause an error (*Message type XXX has no field named YYY*) when the text is attempted to be reparsed. This means they need to be manually deleted from the text when you are editing.</p> 
   <p>When editing a delimited list of messages the editor expects the textual delimiter<code>&gt;--------------------------------next message--------------------------------&lt;</code>&nbsp;between each message. You can add or delete instances of this delimiter to add or remove messages from the list.</p> 
   <h5>Protobuf Viewer - Edit mode</h5> 
   <p>The Protobuf viewer in edit mode shows the current message content in the same structured view described in the&nbsp;<a href="#structured_viewer">Protobuf Structured Viewer</a>section. The one key difference is that all the defined fields for a message are always shown, regardless of whether they are currently populated. The fields which are defined for a message but are not currently populated are shown in&nbsp;<em>italics</em>. This makes it very easy to see what fields may exist for a message and to set values for those fields.</p> 
   <p>The value of any scalar field may be set by simply double-clicking the value column (double-clicking elsewhere on the row will instead open a dialog containing the field value, a feature useful for viewing very large values). Scalar fields can be cleared (turned into a field that is not populated in the message) by right-clicking on the field and selecting&nbsp;<em>Clear value</em>.</p> 
   <p>Message valued fields can similarly be cleared by right-clicking on the field and selecting&nbsp;<em>Clear entire message</em>. Message valued fields that are not populated are treated very similarly to scalar fields, they are shown in italics and populated with their default or implied values. As soon as any child field of an unpopulated message is explicitly set then that message will become a concrete, explicitly-defined value.</p> 
   <p>Repeating fields may have new repeats added to them by right-clicking on the field and selecting&nbsp;<em>Add repeat</em>&nbsp;or by using the&nbsp;<em>Insert before</em>&nbsp;or<em>Insert after</em>&nbsp;commands when you right-click on an existing repeat. There is also&nbsp;<em>Delete repeat</em>&nbsp;option.</p> 
  </div>
 </body>
</html>