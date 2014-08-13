---
layout: post
title: "修改ibus的颜色"
description: ""
category:
tags: ["ibus", "输入法"]
---

看久了千篇一律的ibus输入法的输入框总会有点腻了的感觉

so 稍微hack了一下代码改了一下颜色~


原版ibus-pinyin

![原版ibus-pinyin](/images/2010-09-13-change-ibus-color-1.png)

改版ibus-pinyin

![改版ibus-pinyin](/images/2010-09-13-change-ibus-color-2.png)

改版ibus-sunpinyin

![改版ibus-sunpinyin](/images/2010-09-13-change-ibus-color-3.png)

改版ibus-cloud-pinyin

![改版ibus-cloud-pinyin](/images/2010-09-13-change-ibus-color-4.png)

这下有了点fcitx的感觉了吧~~~

只是自己修改玩玩 所以没有深入研究代码

{% highlight diff linenos %}
*** candidatepanel.py	2010-09-13 14:08:16.000000000 +0800
--- /home/yangguang/Desktop/candidatepanel.py	2010-09-13 12:26:28.000000000 +0800
***************
*** 87,98 ****
          for i in range(0, 16):
              label1 = Label("%c." % ("1234567890abcdef"[i]))
              label1.set_alignment(0.0, 0.5)
-             label1.modify_fg(gtk.STATE_NORMAL, gtk.gdk.Color("#013BC5"))
              label1.show()
  
              label2 = Label()
              label2.set_alignment(0.0, 0.5)
-             label2.modify_fg(gtk.STATE_NORMAL, gtk.gdk.Color("#69A0EC"))
              label1.show()
  
              if self.__orientation == ibus.ORIENTATION_VERTICAL:
--- 87,96 ----
***************
*** 100,110 ****
                  label2.set_property("xpad", 8)
                  ebox1 = EventBox()
                  ebox1.set_no_show_all(True)
-                 ebox1.modify_bg(gtk.STATE_NORMAL, gtk.gdk.Color("#FFFFFF"))
                  ebox1.add(label1)
                  ebox2 = EventBox()
                  ebox2.set_no_show_all(True)
-                 ebox2.modify_bg(gtk.STATE_NORMAL, gtk.gdk.Color("#FFFFFF"))
                  ebox2.add(label2)
                  self.__vbox1.pack_start(ebox1, False, False, 2)
                  self.__vbox2.pack_start(ebox2, False, False, 2)
--- 98,106 ----
***************
*** 116,122 ****
                  hbox.pack_start(label2, False, False, 1)
                  ebox = EventBox()
                  ebox.set_no_show_all(True)
-                 ebox.modify_bg(gtk.STATE_NORMAL, gtk.gdk.Color("#FFFFFF"))
                  ebox.add(hbox)
                  self.pack_start(ebox, False, False, 4)
                  self.__candidates.append((ebox,))
--- 112,117 ----
***************
*** 148,159 ****
              if i == focus_candidate and show_cursor:
                  if attrs == None:
                      attrs = pango.AttrList()
!                 #color = self.__labels[i][1].style.base[gtk.STATE_SELECTED]
!                 #print color
                  end_index = len(text.encode("utf8"))
!                 #attr = pango.AttrBackground(color.red, color.green, color.blue, 0, end_index)
!                 #attrs.change(attr)
!                 color = gtk.gdk.Color("#FF55FF")
                  attr = pango.AttrForeground(color.red, color.green, color.blue, 0, end_index)
                  attrs.insert(attr)
  
--- 143,153 ----
              if i == focus_candidate and show_cursor:
                  if attrs == None:
                      attrs = pango.AttrList()
!                 color = self.__labels[i][1].style.base[gtk.STATE_SELECTED]
                  end_index = len(text.encode("utf8"))
!                 attr = pango.AttrBackground(color.red, color.green, color.blue, 0, end_index)
!                 attrs.change(attr)
!                 color = self.__labels[i][1].style.text[gtk.STATE_SELECTED]
                  attr = pango.AttrForeground(color.red, color.green, color.blue, 0, end_index)
                  attrs.insert(attr)
  
***************
*** 229,236 ****
          self.__moved_cursor_location = None
  
          self.__recreate_ui()
-         
-         self.__viewport.modify_bg(gtk.STATE_NORMAL, gtk.gdk.Color("#FFFFFF"))
  
      def __handle_move_end_cb(self, handle):
          # store moved location
--- 223,228 ----
***************
*** 251,257 ****
  
          # create aux label
          self.__aux_label = Label(self.__aux_string)
-         self.__aux_label.modify_fg(gtk.STATE_NORMAL, gtk.gdk.Color("#EB50A0"))
          self.__aux_label.set_attributes(self.__aux_attrs)
          self.__aux_label.set_alignment(0.0, 0.5)
          self.__aux_label.set_padding(8, 0)
--- 243,248 ----
***************
*** 526,532 ****
      table = ibus.LookupTable()
      table.append_candidate(ibus.Text("AAA"))
      table.append_candidate(ibus.Text("BBB"))
-     table.append_label(ibus.Text("CCC"))
      cp = CandidatePanel()
      cp.show_all()
      cp.update_lookup_table(table, True)
--- 517,522 ----
{% endhighlight %}
