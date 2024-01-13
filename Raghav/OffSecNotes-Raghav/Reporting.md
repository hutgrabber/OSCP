### Consider the Objective
Take into account the objective of the assessment. What did you set out to accomplish? Is there a single, specific statement you hope to make in the report? Many inexperienced penetration testers get caught up in the technical aspects of an assessment and the skills necessary to pull them off, but a penetration test is never an opportunity to simply show off. Keep the initial objective in mind as you begin writing the report.

Organize your content to build a report that will resonate the most with your audience. We highly recommend writing an outline before starting. You can do this quickly and easily by creating section headers, without the actual content or explanation. This will help you avoid repeating yourself or leaving out critical information. It can also help you more easily get past the dreaded "writer's block".

### Consider the Audience
Think about who will be reading and acting on the information you've included in the report. What does your audience hope to learn from it? Who are they? In most cases, people with vastly different levels of technical knowledge will read your report. Try to write something to satisfy each potential reader of the report. Practically speaking, this means writing your report in sections that address the needs of different audiences.

Let's spend a moment talking a bit more about the audience.

You might expect high-level executives in a company to read some parts of the report. In most cases these executives do not have the time or desire to read all of the highly technical details of the attack. For this reason, most reports start with an Executive Summary. The Executive Summary should be a short (no more than two pages), high-level explanation of the results and the client's overall security posture. Since it is likely the only part they will ever read, make sure you tailor this section and the language for the executives specifically.

There will also be a team of more technical professionals who will read your report in greater detail. The rest of the report should cater to them, and will include all the gory details of the carnage you inflicted upon the target network.

### Consider What to Include
More specifically, it's helpful to think about what **not** to include. Keep in mind that your readers will want to address the issues you discovered, so all the content that you include should be relevant and meaningful. A bloated report with too much tangential or irrelevant information just makes reading and understanding difficult for your audience. Don't include filler material just to make the report look longer.

Here are four quick pointers on what to include and what to leave out:

1.  **DO NOT** include pages and pages of a tool output in your report unless it is absolutely relevant. Consider Nmap's output. There is no reason for you to include every single line from the output in your report as it does not add anything of value. If you have a point that you are trying to make, for example a very high number of SNMP services exposed on publicly accessible hosts, then use the –oG flag and grep out only those hosts with open SNMP ports.
    
2.  Make use of screenshots wisely. The same rule applies as with the rest of the content you add to your report. Use a screenshot to make a point, not just to show awesome meterpreter output. For example, say you got root on a Linux host. Rather than displaying 15 screenshots of various directory listings only a root user could access, just include a single screenshot of the whoami command output. A technically savvy reader may only need this one thing to understand what you have achieved.
    
3.  Include extra materials as additional supporting documents. If you have content that will drive up the page count but not be interesting to your entire audience, consider providing additional supporting documents in addition to the report. The readers who need this information can still inspect the supporting documentation and the quality of the report won't suffer.
    
4.  Perhaps most importantly, refer back to the objective of the assessment. Think about the point you are trying to make as it relates to the objective and about how each piece of information will or will not reinforce that point.

### Consider the Presentation
The presentation of content is just as critical as the content itself. More than anything, a command of language is absolutely crucial. While we understand that for many of our students, English is not their native language, it is still important to try to write coherent sentences that flow smoothly and logically. In this case, it is important to "Try Harder" and do your best, focusing on making points that are simple and easy to understand.

Additionally, you may want to keep the following in mind:

1.  Be consistent. Watch out for inconsistencies in things like spacing, heading styles, font selection, and so on. Misaligned and inconsistent paragraphs or titles look unprofessional and sloppy.
    
2.  Spellcheck, spellcheck, spellcheck! This one is pretty self-explanatory. Their != There, Your != You're
    

These pointers should give you a general idea of how to write a professional-looking and coherent report that clearly delivers the intended message. Ultimately, the report is the product you are delivering to the client. Make sure it represents you and your work properly and professionally.

### The PWK Report
After you've completed the course lab guide and videos, you will be conducting a full-fledged penetration test inside our PWK VPN lab network. It's not mandatory to report on this practice penetration test, but it might be beneficial to you as a useful way to practice an important skill that you will use throughout your career.

If you do opt to write and submit your lab report, you will need to document the course exercises throughout this lab guide unless noted otherwise. You can add these as an appendix to your final report that you will submit after completing the certification exam.

The final documentation should be submitted as a formal penetration test report. Your report should include an executive summary, as well as a detailed rundown of all machines (not including your dedicated client machines). Detailed information regarding the reporting requirements for the course, including templates and a sample report is available on our support site.[1](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/penetration-testing-with-kali-linux:-general-course-information/reporting/the-pwk-report#fn1)

In addition to the optional VPN lab network penetration test report, students opting for the OSCP certification must submit an exam penetration test report. That report should clearly demonstrate how they successfully achieved the certification exam objectives. This final report must be sent back to our Certification Board in PDF format no more than 24 hours after the completion of the certification exam.

Students planning to claim CPE credits prior to having passed the OSCP certification exam will need to write and submit a report of the VPN lab network and include the course exercises as an appendix.

[https://help.offensive-security.com/hc/en-us/articles/360046787731-Penetration-Testing-with-Kali-Linux-Reporting](https://help.offensive-security.com/hc/en-us/articles/360046787731-Penetration-Testing-with-Kali-Linux-Reporting) [↩︎](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/penetration-testing-with-kali-linux:-general-course-information/reporting/the-pwk-report#fnref1)

### Taking Notes
Information is key, so taking and keeping organized notes is vital. This goes for the PWK course, the corresponding OSCP exam, and even penetration testing in general.

The level of detail in your notes is up to you. We recommend that you document **everything** to start with. This includes all of the console output, as well as screenshots of key events. It's better to have too much than to repeat material in order to fill in gaps.

Being organized at the outset will pay off in the long term. If you need to return to your notes for any reason in a few weeks, months, or even years, organization will enable you to quickly locate the information you need. Developing good documentation skills will also allow you to quickly find that long command that you used to exploit a given machine several days before, should you ever need to re-exploit it, or cross-reference users during post-exploitation after having successfully compromised each target machine.

Over time, you will start to generate rough templates and formats for your notes. As a result, your notes layout and detail will differ between the start and the end of the course. It is common for us to hear students comment about how much they are missing certain pieces of information at the start, and how they have to go back to the "early targets" to collect it.

Aim to collect as much information from a target as possible. This will allow you to generate a complete report even if you do not have access to the lab. Having good, detailed notes will be especially useful during the post-exploitation phase in the labs, as having certain pieces of information readily available should help you find clear links between lab machines, and so forth. A good documentation process will save you considerable time and a few headaches as well.

#### Setup & Tips
The key to good note-taking is being able to collect as much information as possible and to have it readily accessible. The amount of information may change over time, and so may your process for quickly finding what you need.

You also need to be aware of where the information is being stored--is it local or remote? Is it encrypted? Is there any sensitive information that is part of your notes? If so, consider the possibility that your information (or worse, your client's) could fall into the wrong hands.

To start out, we highly recommend that you capture and document everything. Certain tools support writing their output to a file, and some of them even have reporting capabilities. Capturing your terminal output and then combining it with your personal notes can also be helpful sometimes. Make sure to annotate, highlight important sections, and write down anything you might deem relevant. Keep in mind that sometimes a screenshot is worth a thousand words, so make sure you take them as well.

#### Note Taking Tools
There are a number of note taking tools you can choose from such as OneNote[1](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/penetration-testing-with-kali-linux:-general-course-information/reporting/taking-notes#fn1) (Windows/macOS), DayOne[2](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/penetration-testing-with-kali-linux:-general-course-information/reporting/taking-notes#fn2) (macOS) or Joplin[3](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/penetration-testing-with-kali-linux:-general-course-information/reporting/taking-notes#fn3) (MacOS/Windows/Linux) etc. You can also opt to use something like MDwiki,[4](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/penetration-testing-with-kali-linux:-general-course-information/reporting/taking-notes#fn4) a markdown-based wiki that allows you to write in markdown and then render the output in HTML.

Regardless of your preferred tool, the best way to go about collecting RAW output is to set up some type of logging and forget about it (until it is needed). This way the output is automatically saved and you do not have to worry about remembering to return to your notes. There are a few ways for all output displayed to a terminal to be saved, some of which include:

-   _script_: Once executed, all output (including bash's color & backspaces) is saved to a file, which can be replayed at any time.
    
-   _terminator_: An alternate terminal emulator that has various features and plugins, such as Logger (save all output to a text file) and TerminalShot (take a screenshot from within the terminal).
    

**NOTE**: Piping the output (>) or using tee is also an option, but you have to use them for each command, so you will have to remember to run them every time.

To deal with the volume of information gathered during a penetration test, we like to use a multipurpose note-taking application to initially document all of our findings. Using such an application helps both in organizing the data digitally as well as mentally. Once the penetration test is over, we can use the interim documentation to compile the full report.

It doesn't matter which program you use for your interim documentation as long as the output is clear and easy-to-read. Get used to documenting your work and findings. It is the only professional way to get the job done!

#### Backups

There are two types of people: those who regularly back up their documentation, and those who wish they did. Backups are often thought of as insurance. You never know when you're going to need it until you do! As a general rule, we recommend that you backup your documentation regularly. Keep your backups in a safe place. You certainly don't want them to end up in a public git repo or the cloud!

Documentation should not be the only thing you back up. Make sure you back up important files on your Kali VM, take appropriate snapshots if needed, and so on. It's always best to err on the side of caution.
