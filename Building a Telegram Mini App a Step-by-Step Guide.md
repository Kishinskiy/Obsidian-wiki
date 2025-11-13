
Have you ever wanted to reach millions of users without them having to download your app? That’s exactly what Telegram Mini Apps offer. With over 800 million monthly active users, Telegram provides an incredible platform to deploy your applications directly within their ecosystem.

Did this guide on building Telegram Mini Apps help you? Show your support with a few claps 👏 and follow me for more technical tutorials! I’d love to hear your suggestions for future topics in the comments below — together we can build amazing things! 🚀

## Why Build a Telegram Mini App?

- **Instant Access**: Reach millions of users without app store barriers  
- **Native Integration**: Seamless experience within Telegram’s interface  
- **Real-time Capabilities**: Perfect for collaborative applications  
- **Zero Installation**: Users can start using your app immediately

During a recent project, I developed a Telegram Mini App and encountered several challenges due to limited documentation. I decided to share what I have learned from it.

In this comprehensive guide, we’ll build a collaborative task board for Telegram groups from scratch. You’ll learn how to:  
- Set up a Next.js project with TypeScript  
- Integrate Firebase for real-time data management  
- Deploy your Mini App to production  
- Handle common pitfalls and their solutions

## Understanding the Project Scope

We’re building a task management system that allows Telegram group members to:  
- Create and manage tasks together  
- Track task completion status  
- Delete completed tasks  
- All in real-time, right within Telegram

## **Project Architecture**

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*4Gvi_RFiXC2EbsQdOF-okQ.png)

Interactions between the different components

1. **Telegram Mini App** serves as the entry point for users, providing seamless integration with Telegram’s native interface. It offers direct access to the web app features, allowing users to interact with the application without leaving the Telegram environment.
2. **Next.js Web App** is the core of our application, handling the main application logic with server-side rendering capabilities. Built with TypeScript for enhanced maintainability, it provides the full feature set and manages all user interactions.
3. **Firebase** acts as our central data management system, providing real-time database capabilities and handling authentication. It ensures data synchronization across all components and offers robust cloud functions for backend operations.
4. **Telegram Bot** manages automated notifications and handles direct communication with Firebase. It processes commands and provides additional functionality through the Telegram bot interface, complementing the Mini App experience.

> 💡 **Pro Tip**: This tutorial assumes basic knowledge of React and TypeScript. If you’re new to these technologies, consider reviewing their fundamentals first.

Let’s dive in and build something amazing together!

# The Mini App

We are going to build a collaborative board tasks for Telegram groups. This Mini App will allow group members to create, assign, and manage tasks together, all within the Telegram interface.

## Project setup

We first create a NextJs project and install the needed packages:

npx create-next-app@latest task-board-telegram  
cd task-board-telegram  
npm install @telegram-apps/sdk-react firebase @headlessui/react @heroicons/react  
npm install telegraf dotenv

# Building the Web App

After setting up our project, let’s create a simple task board that will integrate with Telegram groups. We are now going through each file. If you don’t want to go through, you can clone the GitHub repository through this link [https://github.com/Tanguyvans/task-board-telegram](https://github.com/Tanguyvans/task-board-telegram)

## 1. Firebase Configuration

Before we can use Firebase in our application, we need to set it up properly:

a) Create a Firebase Project:  
- Go to [Firebase Console]([https://console.firebase.google.com/](https://console.firebase.google.com/))  
- Click “Add Project”  
- Name your project (e.g., “task-board-telegram”)

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*42pDBVJ6I3P5Yk0nWwPV7Q.png)

Create a Firebase project

b) Enable Firestore:  
- In Firebase Console, go to “Firestore Database”  
- Click “Create Database”  
- Choose “Start in test mode” for development  
- Select a location closest to your users

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*-8qgk3EJz-uscJriUTQZpg.png)

Create a Firestore database

c) Get Firebase Configuration:  
- Go to Project Settings (⚙️ icon)  
- Under “General” tab, scroll to “Your apps”  
- Click the web icon (</>)  
- Register your app with a nickname  
- Copy the firebaseConfig object

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*rcyRLzR35EMEruDrFf0pxA.png)

Get Firebase configuration

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*Ggt6YF-WRhCe8fC8_vhtJA.png)

d) Set up Environment Variables:  
Create a `.env.local` file in your project root:

NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key  
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com  
NEXT_PUBLIC_FIREBASE_DATABASE_URL=https://your_project.firebaseio.com  
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id  
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com  
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id  
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id  
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=your_measurement_id

First, we need to set up Firebase. Create a new file `app/lib/firebase.ts`:

'use client';  
  
import { initializeApp } from "firebase/app";  
import { getFirestore, Firestore } from "firebase/firestore";  
  
const firebaseConfig = {  
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,  
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,  
  databaseURL: process.env.NEXT_PUBLIC_FIREBASE_DATABASE_URL,  
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,  
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,  
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,  
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,  
  measurementId: process.env.NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID  
};  
  
// Initialize Firebase only on client side  
let app;  
let db: Firestore;  
  
if (typeof window !== 'undefined') {  
  app = initializeApp(firebaseConfig);  
  db = getFirestore(app);  
}  
  
export { db };

## 2. Creating the Task Interface

Create `app/types/task.ts`:

export interface Task {  
    id: string;  
    title: string;  
    completed: boolean;  
    createdAt: Date;  
    groupId: string;  
  }

## 3. Building the Components

Create `app/components/TaskForm.tsx`:

"use client";  
  
import { useState } from 'react';  
import { addDoc, collection } from 'firebase/firestore';  
import { db } from '@/app/lib/firebase';  
  
interface TaskFormProps {  
  groupId: string;  
}  
  
export default function TaskForm({ groupId }: TaskFormProps) {  
  const [title, setTitle] = useState('');  
  const [error, setError] = useState<string | null>(null);  
  
  const handleSubmit = async (e: React.FormEvent) => {  
    e.preventDefault();  
    if (!title.trim() || !db) return;  
  
    try {  
      await addDoc(collection(db, 'tasks'), {  
        title: title.trim(),  
        completed: false,  
        createdAt: new Date(),  
        groupId,  
      });  
  
      setTitle('');  
      setError(null);  
    } catch (err) {  
      console.error('Error adding task:', err);  
      setError('Failed to add task');  
    }  
  };  
  
  return (  
    <form onSubmit={handleSubmit} className="space-y-4">  
      <input  
        type="text"  
        value={title}  
        onChange={(e) => setTitle(e.target.value)}  
        placeholder="Add a new task"  
        className="w-full px-4 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"  
      />  
      <button  
        type="submit"  
        className="w-full bg-blue-500 text-white py-2 rounded-md hover:bg-blue-600 transition-colors"  
      >  
        Add Task  
      </button>  
    </form>  
  );  
}

Create `app/components/TaskItem.tsx`:

"use client";  
  
import { useState } from 'react';  
import { updateDoc, doc, deleteDoc } from 'firebase/firestore';  
  
import { db } from '@/app/lib/firebase';  
import { Task } from '@/app/types/task';  
  
interface TaskItemProps {  
  task: Task;  
}  
  
export default function TaskItem({ task }: TaskItemProps) {  
  const [isCompleted, setIsCompleted] = useState(task.completed);  
  
  const toggleComplete = async () => {  
    setIsCompleted(!isCompleted);  
    await updateDoc(doc(db, 'tasks', task.id), {  
      completed: !isCompleted,  
    });  
  };  
  
  const deleteTask = async () => {  
    await deleteDoc(doc(db, 'tasks', task.id));  
  };  
  
  return (  
    <li className="flex items-center justify-between p-4 bg-white rounded-lg shadow">  
      <div className="flex items-center space-x-4">  
        <input  
          type="checkbox"  
          checked={isCompleted}  
          onChange={toggleComplete}  
          className="form-checkbox h-5 w-5 text-blue-600"  
        />  
        <span className={isCompleted ? 'line-through text-gray-500' : ''}>  
          {task.title}  
        </span>  
      </div>  
      <button  
        onClick={deleteTask}  
        className="text-red-500 hover:text-red-700"  
      >  
        Delete  
      </button>  
    </li>  
  );  
}

Create `app/components/TaskList.tsx`:

"use client";  
  
import { useState, useEffect } from 'react';  
import { collection, query, where, onSnapshot } from 'firebase/firestore';  
import { db } from '@/app/lib/firebase';  
import TaskItem from '@/app/components/TaskItem';  
import { Task } from '@/app/types/task';  
  
interface TaskListProps {  
  groupId: string;  
}  
  
export default function TaskList({ groupId }: TaskListProps) {  
  const [tasks, setTasks] = useState<Task[]>([]);  
  
  useEffect(() => {  
    const q = query(  
      collection(db, 'tasks'),  
      where('groupId', '==', groupId)  
    );  
      
    const unsubscribe = onSnapshot(q, (querySnapshot) => {  
      const taskList: Task[] = [];  
      querySnapshot.forEach((doc) => {  
        taskList.push({ id: doc.id, ...doc.data() } as Task);  
      });  
      setTasks(taskList);  
    });  
  
    return () => unsubscribe();  
  }, [groupId]);  
  
  return (  
    <ul className="space-y-4">  
      {tasks.map((task) => (  
        <TaskItem key={task.id} task={task} />  
      ))}  
    </ul>  
  );  
}

## 4. Deployment on Vercel

Deploying your Telegram Mini App on Vercel is straightforward:

a) Prepare for Deployment:  
- Ensure your code is pushed to GitHub  
- Create a [Vercel account]([https://vercel.com](https://vercel.com/))

b) Deploy the Application:  
- Go to [Vercel Dashboard]([https://vercel.com/dashboard](https://vercel.com/dashboard))  
- Click “New Project”  
- Import your GitHub repository  
- Configure the project:  
— Framework Preset: Next.js  
— Root Directory: ./  
— Build Command: next build  
— Output Directory: .next

c) Environment Variables:  
- In Vercel project settings, go to “Environment Variables”  
- Add all your Firebase environment variables:

NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key  
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com  
NEXT_PUBLIC_FIREBASE_DATABASE_URL=https://your_project.firebaseio.com  
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id  
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com  
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id  
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id  
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=your_measurement_id

d) Deploy:  
- Click “Deploy”  
- Wait for the build to complete  
- Your app will be available at: `[https://your-project.vercel.app`](https://your-project.vercel.app`/)

# Building the Telegram Bot

## 1. Creating the Bot with BotFather

First, we need to create a new bot using Telegram’s BotFather:  
1. Open Telegram and search for [[@BotFather](http://twitter.com/BotFather)]([https://t.me/botfather](https://t.me/botfather))  
2. Send `/newbot` command  
3. Choose a name for your bot  
4. Choose a username (must end in ‘bot’)  
5. **Important**: Save the API token provided, you’ll need it later

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*lPhZds8QBDYAeoOvcfI_UQ.png)

Telegram Bot creation

## 2. Setting Up Bot Commands

Configure the bot commands using BotFather’s `/setcommands`:

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*MxaIwOBDFcW9EM5gDF1aiA.png)

Telegram Bot setting up commands

## 3. Building Bot

Create a new `bot` directory in your project root:

mkdir bot  
cd bot  
mkdir commands types

Update .env.local

NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key  
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com  
NEXT_PUBLIC_FIREBASE_DATABASE_URL=https://your_project.firebaseio.com  
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id  
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com  
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id  
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id  
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=your_measurement_id  
  
BOT_TOKEN= bot_token  
WEBAPP_URL= your_vercel_url

Create `bot/config.ts`:

const dotenv = require('dotenv');  
const path = require('path');  
  
dotenv.config({ path: path.resolve(__dirname, '../.env.local') });  
  
module.exports = {  
  BOT_TOKEN: process.env.BOT_TOKEN,  
  WEBAPP_URL: process.env.WEBAPP_URL  
};

Create `bot/index.ts`:

const { Telegraf } = require('telegraf');  
const { BOT_TOKEN, WEBAPP_URL } = require('./config');  
  
if (!BOT_TOKEN) {  
  throw new Error('BOT_TOKEN must be provided!');  
}  
  
const bot = new Telegraf(BOT_TOKEN);  
  
// Basic commands  
bot.command('start', (ctx: any) => {  
  ctx.reply('Welcome to TaskVaultBot! 🚀\nUse /help to see available commands.');  
});  
  
bot.command('help', (ctx: any) => {  
  ctx.reply(  
    'Available commands:\n' +  
    '/start - Start the bot\n' +  
    '/help - Show this help message\n' +  
    '/webapp - Open the Mini App'  
  );  
});  
  
bot.command('webapp', (ctx: any) => {  
  ctx.reply('Open Web App', {  
    reply_markup: {  
      inline_keyboard: [[  
        { text: "Open App", web_app: { url: WEBAPP_URL || '' }}  
      ]]  
    }  
  });  
});  
  
bot.launch().then(() => {  
  console.log('Bot is running...');  
});  
  
// Enable graceful stop  
process.once('SIGINT', () => bot.stop('SIGINT'));  
process.once('SIGTERM', () => bot.stop('SIGTERM'));

## Running the Bot

Add to your `package.json`:

{  
  "scripts": {  
    "bot:dev": "ts-node bot/index.ts"  
  }  
}

Start the bot:

npm run bot:dev

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*zZQk4ipT6V-hIwfcWFrCtQ.png)

Telegram Bot interactions

## Current Limitations

At this step, you should have a fully running App. However, there are two main issues:

**Group Chat Permissions**: The bot won’t respond to the ‘/webapp’ command in Telegram groups

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*Yea0F4K2ml-KtA9kB_coow.png)

**Missing Group ID**: when opening the Mini App, you will receive a message saying that we did not provide a group ID.

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*HABRWV1n9KQUmG3jmrQing.png)

Telegram Group Chat

# **Handling Group integration**

To make our Web App accessible within Telegram groups, we need to register it as a Mini App:

## Creating a Mini App with BotFather

To make our Web App accessible within Telegram groups, we need to register it as a Mini App:  
— Open [@BotFather](http://twitter.com/BotFather) in Telegram  
— Send the `/newapp` command  
— Select your bot from the list

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*tPuNwvH13OWuyvA1scY5wg.png)

Telegram Creating a New App

— Enter a name for your Mini App  
— Provide your Web App URL (from Vercel deployment)  
— Choose a short name (optional)  
— Upload a profile photo (optional)

Zoom image will be displayed

![](https://miro.medium.com/v2/resize:fit:1400/1*qHIKg0pVAuKjZI_TtWDWdg.png)

Telegram New App url

After completing the setup, BotFather will provide you with a Mini App URL.

## Updating the Code

Update your `.env.local` file with the new Mini App URL:

  
WEBAPP_URL=https://t.me/YourBotName/Home  # Replace with your Mini App URL

Modify the webapp command in `bot/index.ts` to use the Mini App URL:

bot.command('webapp', (ctx: any) => {  
  ctx.reply('Open Web App', {  
    reply_markup: {  
      inline_keyboard: [[  
        { text: "Open App", url: WEBAPP_URL || '' }  
      ]]  
    }  
  });  
});

> 💡 **Pro Tip**: Make sure to restart your bot after updating the environment variables for the changes to take effect.

# Integrating group ID

To make our Mini App work within Telegram groups, we need to pass and handle the group ID correctly. This allows us to manage tasks specific to each group.

## Updating the Bot

First, let’s modify our bot to pass the group’s chat ID to the Mini App:

bot.command('webapp', (ctx: any) => {  
  const chatId = ctx.chat.id;  
  // Encode le chatId en base64  
  const encodedGroupId = Buffer.from(chatId.toString()).toString('base64');  
    
  console.log('Chat ID:', chatId);  
  console.log('Encoded Group ID:', encodedGroupId);  
    
  ctx.reply('Open Web App', {  
    reply_markup: {  
      inline_keyboard: [[  
        { text: "Open App", url: `${WEBAPP_URL}?startapp=${encodedGroupId}` }  
      ]]  
    }  
  });  
});

> 💡 **Note**: We use base64 encoding to safely transmit the chat ID through the URL.

## Handling Group ID in the Web App

Next, we need to update our web app to receive and process this parameter. In Telegram Mini Apps, we can only receive parameters through the `startapp` parameter.

Here’s our updated `page.tsx`:

"use client";  
  
import { Suspense, useEffect, useState } from 'react';  
import Image from "next/image";  
import TaskList from "./components/TaskList";  
import TaskForm from "./components/TaskForm";  
import { useLaunchParams } from "@telegram-apps/sdk-react";  
import dynamic from 'next/dynamic';  
  
// Créer un composant client-only pour le TaskBoard  
const TaskBoardClient = dynamic(() => Promise.resolve(TaskBoard), {  
  ssr: false  
});  
  
function TaskBoard() {  
  const [groupId, setGroupId] = useState<string | null>(null);  
  const [error, setError] = useState<string | null>(null);  
  const [isLoading, setIsLoading] = useState<boolean>(true);  
  const launchParams = useLaunchParams();  
  
  useEffect(() => {  
    const initializeComponent = async () => {  
      try {  
        if (launchParams?.startParam) {  
          const encodedGroupId = launchParams.startParam;  
          try {  
            const decodedGroupId = atob(encodedGroupId);  
            console.log("Decoded Group ID:", decodedGroupId);  
            setGroupId(decodedGroupId);  
          } catch (error) {  
            console.error("Error decoding group ID:", error);  
            setError("Invalid group ID format");  
          }  
        } else {  
          console.log("No start_param available");  
          setError("No group ID provided");  
        }  
      } catch (error) {  
        console.error("Error in initializeComponent:", error);  
        setError("An error occurred while initializing the component");  
      } finally {  
        setIsLoading(false);  
      }  
    };  
  
    initializeComponent();  
  }, [launchParams]);  
  
  if (isLoading) {  
    return <div className="p-8">Loading...</div>;  
  }  
  
  if (error) {  
    return <div className="p-8 text-red-500">{error}</div>;  
  }  
  
  if (!groupId) {  
    return <div className="p-8">Please provide a valid group ID</div>;  
  }  
  
  return (  
    <div className="grid grid-rows-[auto_1fr_auto] min-h-screen p-8 gap-8">  
      <header className="flex items-center justify-between">  
        <Image  
          className="dark:invert"  
          src="/next.svg"  
          alt="Next.js logo"  
          width={100}  
          height={20}  
          priority  
        />  
        <h1 className="text-2xl font-bold">Task Board - Group {groupId}</h1>  
      </header>  
  
      <main className="flex flex-col gap-8">  
        <TaskForm groupId={groupId} />  
        <TaskList groupId={groupId} />  
      </main>  
  
      <footer className="flex justify-center text-sm text-gray-500">  
        Powered by Next.js  
      </footer>  
    </div>  
  );  
}  
  
export default function Home() {  
  return (  
    <Suspense fallback={<div className="p-8">Loading...</div>}>  
      <TaskBoardClient />  
    </Suspense>  
  );  
}

## Key Implementation Details

1. Bot Side:  
— Encodes the group’s chat ID in base64  
— Passes it through the `startapp` URL parameter  
— Logs the IDs for debugging

2. Web App Side:  
— Uses `useLaunchParams` to get the `startapp` parameter  
— Decodes the group ID from base64  
— Handles various states (loading, error, success)  
— Client-side only rendering to avoid SSR issues

# **Result**

When properly integrated, our Mini App now handles group-specific tasks seamlessly. Let’s look at how it works in practice:

## **Command Execution**

Running the `/webapp` command in any Telegram group generates a unique link containing that group’s encoded ID. This allows users to access the Mini App directly from their group chat, maintaining the group context throughout their session.

## **Group-Specific Features**

The Mini App now operates in a fully group-aware mode. All tasks are automatically associated with the current group, ensuring that users only see and manage tasks relevant to their group. Real-time updates are scoped to the specific group, maintaining data isolation between different Telegram groups.

## **Visual Verification**

For transparency and debugging purposes, the group ID appears in the header of the Mini App. This visual indicator helps verify that users are working within the correct group context. As users switch between different groups, they’ll see distinct task lists corresponding to each group’s activities.

Zoom image will be displayed![](https://miro.medium.com/v2/resize:fit:4240/1*8WMmv-FuO3koz1sPCUaY7g.png)

Zoom image will be displayed![](https://miro.medium.com/v2/resize:fit:4244/1*xXp6Kq9_OaC-ybVV5rH67g.png)

Telegram Mini App in different Group chats

> 💡 ****Pro Tip****: Test your Mini App in multiple groups to ensure proper isolation of tasks between groups.

# Conclusion

We’ve successfully built a Telegram Mini App that demonstrates key integration concepts: creating a Mini App, integrating it with Telegram groups, and handling parameters between the bot and web application. This example provides a solid foundation for understanding how Telegram Mini Apps work and how to develop your own.