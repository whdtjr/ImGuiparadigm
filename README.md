THIS IS WIP/DRAFT / [Discuss this article](https://github.com/ocornut/imgui/issues/3815)

### Why

**This is an attempt at explaining what the IMGUI paradigm stands for, and what it could be**.

Please note the [References](#references) section where other articles have been written about this.

Proponent of the IMGUI paradigm have noticed that it was widely misunderstood, over and over.
As of ~~Feb 2021~~ January 2023, even the [IMGUI Wikipedia page](https://en.wikipedia.org/wiki/Immediate_mode_GUI) is **COMPLETELY** off the mark and incorrect. There are reasons for that:
- the acronym is misleading (read below).
- people didn't do a good enough job explaining or documenting what IMGUI means?
- they are different interpretation of what IMGUI means?
- many popular IMGUI implementation have made similar design choices, making it more confusing what is actually the soul and backbone of the IMGUI paradigm vs a set of implementation choices. 
- the naming and success of "Dear ImGui" blurred the line a little bit further (should have used another name).

The acronym is misleading because "immediate-mode" was initially coined as a reference to obsolete graphics API which made it very easy to render contents. Some people still incorrectly assume that IMGUIs are often rendering using those API. 

From now on, IMGUI will stand for "Incredibly Magic Graphics User Interface" ;)

The second purpose of this page should be to make it clear that there is a large space to explore in UI programming.

### The Pitch of Dear ImGui

From [README](https://github.com/ocornut/imgui#the-pitch), notice the four first points:

- "Minimize state synchronization."
- "Minimize state storage on user side."
- "Minimize setup and maintenance."
- "Easy to use to create dynamic UI which are the reflection of a dynamic data set."

While they don't constitute a definition of the IMGUI paradigm, they may give a good intuition of some of the advantages usually associated with the IMGUI paradigm. 

**In order to further clarify this intuition**, we'll provide an example.

Typical RMGUI:
```cpp

// editor.h
// [...] somewhere in a class declaration
MenuItem* m_SaveItem;

// editor.cpp
// [...] somewhere in an init/constructor function
m_SaveItem = Lib_CreateMenuItem(m_ContainerMenu);
m_SaveItem->OnActivated = &OnSave; // Bind action

// [...] somewhere in a shutdown/destructor function
Lib_DestroyItem(m_SaveItem);
m_SaveItem = NULL;
// TODO: Ensure initial dirty state is reflected

// [...] React to item being pressed
void OnSave()
{
    m_Document->Save();
}

// [...] Sync enable state so item can be greyed
// IMPORTANT: Don't forget to call otherwise update menu color won't be correct!
void UpdateSaveEnabledState()
{
    m_SaveItem->SetEnabled(m_Document != NULL && m_Document->m_IsDirty);
}
void SetDocumentDirty(bool dirty) 
{
    if (m_Document)
         m_Document->m_IsDirty = dirty;
    UpdateSaveEnabledState();
}
void SetDocument(Document* document)
{
    m_Document = document;
    UpdateSaveEnabledState();
}
```

Typical IMGUI:
```cpp
// editor.cpp
void UpdateUI()
{
    bool is_save_allowed = (m_Document != NULL && m_Document->m_IsDirty);
    if (Lib_MenuItem("SAVE", is_save_allowed))
        m_Document->Save();
}
```

This is a simple and perhaps exaggerated example provided to get a better intuition about the difference of IMGUI vs RMGUI. There are lots of things to say and criticize about this example. It tends to illustrates the shortcoming of RMGUIs more than it illustrates the shortcomings of IMGUIs. But it should illustrate the core benefit that IMGUIs **tends to facilitate data binding, action binding, and creation/destruction of UI components**. It does facilitate those things because it generally doesn't need them.

### History

The acronym IMGUI was coined by Casey Muratori in this video in 2005:
<BR>  https://www.youtube.com/watch?v=Z1qyvQsjK5Y (see also [blog](https://caseymuratori.com/blog_0001))

_"I’ve also seen lots of people getting into arguments about immediate-mode vs. retained-mode GUI APIs. I think this has something to do with how simple the IMGUI concept is, as it leads people to think they understand it, and then they proceed to get into heated arguments as if they actually know what they’re talking about. I rarely see this problem when I’m talking about anything even mildly complicated, like quaternions."_

Casey's work on formulating and popularizing research on this topic has been pivotal and invaluable. His video and the forums that followed helped people gather around the concept and talk about it. Archive.org has a [2007 Mirror of Molly Rocket's IMGUI forums](https://web.archive.org/web/20070824203105/http://www.mollyrocket.com/forums/viewforum.php?f=10&sid=9680eeedbe87034741d936cbfe319f57), including [first notable post by Casey on 2005-07-05](https://web.archive.org/web/20070824213736/http://www.mollyrocket.com/forums/viewtopic.php?t=134&start=0&sid=cb913629aa8310947c0476848a8824dd). Unfortunately the [2013 Mirror of Molly Rocket's IMGUI forums](https://web.archive.org/web/20131221233224/http://mollyrocket.com/forums/viewforum.php?f=10&sid=b7ce0b0de4493f63edd88d13c99501a9) has index but no actual posts.

The concept has been privately researched before and after. [Thierry Excoffier's Zero Memory Widget](https://web.archive.org/web/20220516003813/https://perso.univ-lyon1.fr/thierry.excoffier/ZMW/) and his [research report](https://web.archive.org/web/20220516004122/https://perso.univ-lyon1.fr/thierry.excoffier/ZMW/rr_2003_03_11.pdf) in 2003 are also very notable although they didn't reach the game development sphere back then.

### Do we need a definition?

_"I kind of wish there was more radical experimentation in this space"_ ([tweet](https://twitter.com/pervognsen/status/1361241939593416705))
<BR>_"But, I need to store a variable therefore it isn't IMGUI anymore!!!"_ (many people)

Nowadays I am starting to believe that the term has caused more harm than benefits, because it suggests there are two camps "IMGUI" vs "RMGUI". The reality is there is a continuum of possibilities over multiple unrelated axises. Many of them haven't been explored enough, as popular IMGUI libraries have been designed for certain needs and not others. Many of them are at reach as extension of existing popular frameworks. Some are likely to only ever exist as part of yet undeveloped IMGUI frameworks.

The existence of those two terms effectively has hindered both discussions and research. The fact that we are even trying to put an official definition to a label is hindering people's imagination about the many possible areas of researches to be done in UI space.

### Half of a definition

@ocornut's attempt for a definition (WIP, 2021)

What we care about:

- IMGUI refers to the API: literally the interface between the application and the UI system.
  - The API tries to minimize the application having to retain data related to the UI system.
  - The API tries to minimize the UI system having to retain data related to the application.
- IMGUI does NOT refer to the implementation. Whatever happens inside the UI system doesn't matter.

This is in comparison with typical RMGUI ("retained-mode UI"):

- RMGUI APIs often have the application retain artifacts from the UI system (e.g. maintain references to UI objects).
- RMGUI APIs often require synchronization mechanisms because the UI system retain more application data.

What it doesn't stands for:

- IMGUI does not mean that the library doesn't retain data.
- IMGUI does not mean that stuff are drawn immediately.
- IMGUI does not mean it needs a continuous loop nor need to refresh continuously.
- IMGUI does not mean it needs to refresh all-or-nothing.
- IMGUI does not mean that states are polled.
- IMGUI does not mean the UI doesn't look "native" or looks a certain way.
- IMGUI does not mean it cannot animate.
- IMGUI does not mean it needs a GPU to render.
- IMGUI does not mean that the library can or cannot support feature x or y.
- IMGUI does not mean that the library is more or less portable.

TODO: Each of those points should be explained with a paragraph. We could also describe how common UI libraries (of all types) stand on a given axis. We could also describe less explored paths and envision what new UI libraries could do.

It's particularly important we develop this section to dissociate the promise (sometimes unfulfilled) vs current implementation. The full-feature bell-and-whistle promise is more likely to be ever fulfilled by a UI library if people understand that it is possible.

### Examples

_TODO_

### Modeless write-up

March 2022:
https://news.ycombinator.com/item?id=30814797

_"Immediate mode is a style of API where important state is kept in user code [editor note: this should be "user side" not "user code"] instead of being retained inside the API implementation. For example, an immediate mode GUI checkbox implementation does not store a boolean value determining whether the checkbox is checked. Instead, user code passes that information as a function parameter whenever the UI needs to be drawn. Even the fact that a checkbox exists on screen is not stored in the GUI library. The checkbox is simply drawn when user code requests it each frame."_

_"Counter intuitively this is often less complicated than the traditional "retained mode" style of GUI libraries because there is no duplication of state. That means no setup or teardown of widget object trees, no syncing of state between GUI objects and application code, no hooking up or removing event handlers, no "data binding", etc. The structure and function of your UI is naturally expressed in the code of the functions that draw it, instead of in ephemeral and opaque object trees that only exist in RAM after they're constructed at runtime. You retain control over the event loop and you define the order in which everything happens rather than receiving callbacks in some uncertain order from someone else's event dispatching code."_

_"Crucially, "immediate mode" is a statement about the interface of the library, not the internals. Common objections of the form "immediate mode GUIs can't support X" or "immediate mode GUIs are inefficient because Y" are generally false and based on a misconception that immediate mode GUIs are forbidden from retaining any internal state at all. It is perfectly normal for an immediate mode library to retain various types of internal state for efficiency or other reasons, and this is fine as long as the state stored in user code remains the source of truth. This can even go as far as internally constructing a whole retained widget tree and maintaining it via React-like tree diffing."_

### Vurtun's write-up

[@vurtun](https://github.com/vurtun) said very eloquently around April 2019:

_"I usually don't like to write about immediate mode UI since there are a lot of preconceived notions about them. Addressing all of them is hard since often that is required since they all are thrown at once. For example a single implementation of a concept does not define a concept (dear imgui is a immediate mode UI not all imgui are like dear imgui)._

_Everything that can be done with "retained" gui can by definition be done by an immediate mode gui (I hate this whole divide at this point). Then we have state. For some reason like andrew already said I often see confusion that immediate mode means stateless. Immediate mode has nothing to do with how or if state is stored. Immediate mode in guis doesn't remove having two data representation. One on the calling side and one in the library. Even dear imgui and nuklear both have internal state they keep track of. Rather at its core immediate mode is about how this state is updated. In classical gui the state of the application was synchronized to the gui state by modification. On the other hand immediate mode is closer to functional programming. Instead of mutating state, all previous state is immutable and new state can only be generated by taking the previous state and applying new changes. Both in dear imgui and nuklear there is very little previous state and most is build up every "frame"._

_Next up handling widget events (press,clicked,...). Immediate mode has nothing to do with events vs polling for input changes. The famously convenient if (button(...)). You can even have retained libraries that use polling instead of events._

_Then we have "immediate mode bundles layouting, input and rendering". Once again only dear imgui and nuklear do this. At this point all my imguis have multiple passes. For example if I cache state for layouting I have (layout, input, render). Any change in the input or render pass will will jump back to layouting. If I don't cache state for layouting I still have two passes (input, render) to prevent glitches. To make sure that performance is not a problem I make sure that only those elements that are actually visible are updated, which is important for datastructures (list, trees). Invisible widgets are not only not rendered but also not layouted or have input handling. In a current implementation they are not even iterated over._

_Another argument I've read is that immediate mode cannot be used with OOP. While I try to stay away from that term since at this point it has taken so many meaning that it is basically worthless, what I hear it in context of is having composability for widgets. Once again just because dear imgui and nuklear don't have does not mean it has to be or even should be that way. For example the imgui we have at work supports composability and just like retain mode UIs you can put any widget inside any other. Want a slider inside a tab you can just compose them (not that it makes sense in this case)._

_Getting to game vs. tool ui. Disclaimer: I spend most of my time on tool ui however I played around with game ui or rather the concept behind it in my immediate/retain mode hybrid (https://gist.github.com/vurtun/c5b0374c27d2f5e9905bfbe7431d9dc0)._

_Personally for me tool and game ui are two completely different use cases. Interestingly at work we actually have a single immediate mode core that is used for both game as well as tool ui and it still works. However our advantage is that our UI designer can code. Immediate mode biggest advantage is that state mutation is extremely simple and easy. However in games all UI (including stuff like animations) outside datastructures likes list are know at compile time or loading time. So most advantages of immediate mode doesn't hold up (outside mentioned datastructures). Interestingly for datastructures you can even have an immediate mode API inside a retained state UI. So having the best of both worlds by having a hybrid._

_Finally state handling. For games this is easier since most state is known beforehand and can be reserved/preallocated. For tool ui it is a bit more complicated. First of it is possible to have state per widget. That is what we currently do at keen core. The state itself is lifetime tracked and freed if not used anymore. However what is actually possible as well is having a concept of an window/view and managing state outside the library. By that I mean a tabbed window with content. Like for example a log view or scene view storing all needed state for the UI and the actually data needed (log entries for the log view). Instead of trying to do state handling on the widget level it makes a lot more sense to have state management inside the view itself, since unlike the actually widgets the view itself has an easy to track lifetime. Just to clarify the view in the library API would just an handle while the actual view implementation would store the state._

_A lot in ui depends on the problem on hand so I feel there is no silver bullet. But what is actually great about UI is that it is possible to write a version fitting the problem. I hope if anything is taken from this rant is that "imgui" currently entangles a lot of concepts that however don't define it. So everytime it does not work out immediate mode as a grand concept itself is tossed aside instead of reevaluating the parts that did not work out."_

### References

- [Johannes 'johno' Norneby's article](http://www.johno.se/book/imgui.html), 2007.
- [A presentation by Rickard Gustafsson and Johannes Algelind](http://www.cse.chalmers.se/edu/year/2011/course/TDA361/Advanced%20Computer%20Graphics/IMGUI.pdf), 2011.
- [Jari Komppa's tutorial on building an IMGUI library](http://iki.fi/sol/imgui/), @jarikomppa, 2006.
- [Casey Muratori's original video that popularized the concept](https://mollyrocket.com/861), 2005.
  - [First notable post by Casey on 2005-07-05](https://web.archive.org/web/20070824213736/http://www.mollyrocket.com/forums/viewtopic.php?t=134&start=0&sid=cb913629aa8310947c0476848a8824dd). 
  - [2007 Mirror of Molly Rocket's IMGUI forums](https://web.archive.org/web/20070824203105/http://www.mollyrocket.com/forums/viewforum.php?f=10&sid=9680eeedbe87034741d936cbfe319f57) (with posts).
  - [2013 Mirror of Molly Rocket's IMGUI forums](https://web.archive.org/web/20131221233224/http://mollyrocket.com/forums/viewforum.php?f=10&sid=b7ce0b0de4493f63edd88d13c99501a9) (index only, missing posts).
- [Sean Barrett's article in Game Developer Magazine](https://archive.org/details/GDM_September_2005) (Page 34), September 2005.
- [Sean Barrett's IMGUI page and toolkit](http://silverspaceship.com/inner/imgui), @nothings, 2005.
- [Nicolas Guillemot's CppCon'16 flash-talk about Dear ImGui](https://www.youtube.com/watch?v=LSRJ1jZq90k), 2016.
- [Thierry Excoffier's ZMV (Zero Memory Widget)](http://perso.univ-lyon1.fr/thierry.excoffier/ZMW/), 2004.
- [Unity's IMGUI](https://docs.unity3d.com/Manual/GUIScriptingGuide.html)
