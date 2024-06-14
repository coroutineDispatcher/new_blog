---
title: "What is a HAL anyways?"
seoTitle: "What is HAL in Android"
seoDescription: "In this article we will learn what is a HAL - hardware abstraction layer and why it exists."
datePublished: Fri Jun 14 2024 18:22:54 GMT+0000 (Coordinated Universal Time)
cuid: clxf0nz98000c09l6ayjsa5jt
slug: what-is-a-hal-anyways
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1718389325134/42c77a8a-dc6d-4311-bc7a-5d8390af7392.jpeg
tags: cpp, c, linux, java, android, hal, kotlin, linux-kernel, aosp, platform-engineering, platform-architecture, hardware-abstraction-layer, android-platform

---

Many of you already know that Android is nothing but a smartphone Linux distro. It is essentially a variant of GNU/Linux. When examining the Android platform architecture layers, we notice something very familiar to all Linux distributions: the Linux Kernel.

The image below is sourced from the official Google documentation:

![Android Platform architecture. Image taken from Google: https://developer.android.com/guide/platform](https://cdn.hashnode.com/res/hashnode/image/upload/v1718385474115/39d7b56f-c559-4f41-878e-44f38b46dc87.png align="center")

At the core of the Android platform architecture is the kernel. It is essentially a software layer that uses a set of complex instructions to interact with the hardware. Everything in this layer is written in C. If you come from the world of Microsoft Windows, the term "driver" will be familiar to you. This is exactly what happens in the kernel.

A driver is a piece of software consisting mainly of basic commands to communicate with a specific piece of hardware. For example, the audio driver is used when a user talks on the phone or listens to music, and the camera driver is used when the user takes pictures or records videos. As mentioned earlier, everything is more complex because it is written in C.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718386324811/5099b42e-8bde-4359-af8a-2f6b4165480c.png align="center")

However, if we move up a layer, we notice something very similar: the Hardware Abstraction Layer (HAL). The HAL is an abstraction layer built in C and C++ that interacts with the drivers of the lower layer, e.i the kernel. If we look at the image below, we can see an unusual similarity with what the kernel offers:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718386687629/d48b4740-2477-4be2-9747-b6f6d5933c01.png align="center")

One might ask: Why do we need a layer that simply wraps some C drivers into C++ interfaces? Can't we just access the kernel directly from the Native Libraries or the Android Runtime, since they are also mostly in C (and a little bit of Java)?

**The technical stuff**

The primary purpose of the HAL is to provide a standardized interface for the Android framework to interact with the hardware (through the kernel). This allows developers to write code that works across various devices without needing to know the specifics of each hardware component.

Understanding different Linux kernel configurations for various hardware components can be quite complex. Android devices have many such components, each requiring specific handling. The HAL abstracts these details, allowing the Android system to interact with the hardware uniformly, regardless of the underlying kernel differences.

**Business and leagal reasons**

As usual, these aspects can be more complicated than the computer science itself. The GNU/Linux kernel is released under the GNU General Public License (GPL). This license employs a concept known as Copyleft, which requires that any modifications to the kernel must also be distributed under the same GPL license. This ensures that the modified code remains open source.

You can read more about the license here: [GNU General Public License](https://www.gnu.org/licenses/gpl-3.0.en.html)

![ [GPLv3 Logo] ](https://www.gnu.org/graphics/gplv3-127x51.png align="left")

***But I don't want to Open Source my Android code. I want to stay in the competition.***

For instance, Samsung develops custom modifications and enhancements to the Android OS for their devices, such as the Galaxy series (specifically for the camera app). Because Android is built on the Linux kernel, Samsung must release any kernel modifications as open source to comply with the GPL licence. This is why Samsung publishes their modified kernel source code on their open-source release center website. However, they can keep their custom user interface and specific applications proprietary, as these do not modify the kernel. This is all possible thanks to the Hardware Abstraction Layer (HAL).

In my humble opinion, the introduction of the HAL layer is driven more by business considerations than technical ones. A C++ developer would not mind working in C either. OEMs are exploring new business opportunities, and the use of Android is flourishing. Many automotive giants have also embraced Android because of this. Their developers can program in the HAL without needing to modify the Linux kernel at all.

Hopefully, this article provides a clear understanding of what a Hardware Abstraction Layer (HAL) is and why it exists. As technology continues to advance, the HAL will play a crucial role in maintaining compatibility and simplifying development across diverse hardware platforms. This ensures that both OEMs and developers can innovate while adhering to necessary open-source requirements.