> **CHAPTER 1: Typescript - What and why?**

Typescript (originally) is a superset of the javascript programming language. It's purpose is to provide static typings, allowing developers to be more confident in the code that they write.

The static type safety provided by typescript amounts to more work upfront in exchange for FAR less unpredictable errors at runtime.

**Q:** *Wait, luau support types. Why would i choose typescript?*

**A:** While luau does support partial static typing, it is a gradually typed language. This basically means that static typings can be half-baked into a codebase and left to error at runtime. 

Overall, Luau allows us to be lazy with our type annotations. See the following example:

```lua
-- Proper Type Annotations
function Append(Str1: string, Str2: string): string
    return Str1 .. Str2
end

Append(1, "Hello") -- If in strict mode, this will throw a type error.

-- No Type Annotations
function AddOne(Number) 
    return Number + 1
end

AddOne("Hello!") -- No issues until you run the game.
```

Typescript on the other hand will not even allow us to compile if there is a single type error within our codebase.

Here is the append method written in typescript:

```ts
function Append(Str1: string, Str2: string): string {
    return Str1 + Str2;
}

// TS will refuse to compile, because this method should never take anything other than a string.
Append(1, "Hello");
```

*New to typescript? Check out this* [typescript crash course](https://learnxinyminutes.com/docs/typescript/).


> **CHAPTER 2: SETUP**

**Prerequisites**

1. External Editor
    - This tutorial assumes you are using vscode, but most editors will work fine.
2. [Node.js 14+](https://nodejs.org/en/) Installed
3. [Rojo](https://rojo.space/docs/v7/getting-started/installation/) plugin or *preferably* [Rojo CLI](https://rojo.space/docs/v7/getting-started/installation/) 

**Getting Started**

Osiris has made the documentation for setup very clear and concise. For that reason, I'll link the setup guide here. **Follow it till step three, then return back.** https://roblox-ts.com/docs/setup-guide

Once you have setup the project using `rbxtsc init game`, you should see some new folders appear.

Awesome! You've setup your first project. If you have used rojo before, you will be familiar with the layout within the `src` folder. 

You will also notice a `out` folder that mirrors the layout of `src`. This folder is the output of the typescript transpiler, and contains luau code.

**Helpful Tools**

Let's add some packages to improve the developer experience. Follow these steps:

1. Open your integrated terminal with `Ctrl+Shift+tilde(~)`
2. Type `npm i` then `Enter` to install any required packages.
3. Type `npm i -g concurrently` then `Enter`. (Install the package)
   1. This will install a tool to your global registry that allows us to run two shell commands at the same time. This will be useful later.
4. Install `npm i @rbxts/services`
   1. This installs a module within our project that automatically imports roblox services. Super handy!

**Woo!** We now have our basic packages installed! Let's continue:

1. Open the `package.json` in the root of your project.
2. Locate the `scripts: []` field.
3. Copy and paste this inside the scripts array: `"dev": "concurrently \"rbxtsc -w\" \"rojo serve\""`
4. Save the file and close it.

What we have just done is create a new command that runs the typescript compiler (`rbxtsc -w`) in watch mode, and `rojo serve` at the same time.

Let's give our new command a try. Go ahead and run `npm run dev` within your terminal. If all is working well, yous should see the following:

```sh
roblox-ts/FPS Framework > npm run dev

> flamework-template@1.0.0 dev
> concurrently "rbxtsc -w" "rojo serve"

[1] Rojo server listening:
[1]   Address: localhost
[1]   Port:    34872
[1] 
[1] Visit http://localhost:34872/ in your browser for more information.
[0] [11:27:00 AM] Starting compilation in watch mode...
[0] 
[0] [11:27:01 AM] Found 0 errors. Watching for file changes.
[0] 
```

If you see this, then you are ready to get started! Your typescript code will be automagically converted to luau on save, and those changes will sync into studio via rojo.

*You will need to run this command once whenever you being work on your project.*

Finally, go ahead and connect to rojo using the studio plugin.


> **CHAPTER 3: Usage**

Let's do an example! Let's make a basic killbrick.

(this example is showcased on the roblox-ts homepage)

First, connect rojo to your project. If you are using vscode, use `Ctrl/Cmd+Shift+P` to open the command palette. Type Rojo, and click the `Open Menu` option. Then, click the item with a play icon at the bottom.

Head over to `src/server` and create a new file titled `index.server.ts`. This will compile to a server script in game.

1. Begin to type `CollectionService` on line 2.
   - You should see a auto-completion for CollectionService. Hit enter.

You should now see the following line appear at the top of the file.
```ts
import { CollectionService } from "@rbxts/services";
```

This is an example of roblox-ts auto imports!

Let's now loop through each instance with the `Lava` tag, and check if it's a base part.
```ts
for (const obj of CollectionService.GetTagged("Lava")) {
    if (obj.IsA("BasePart")) {

    }
}
```

Great! Now this is where type script really shines. Lets write out some kill logic:

```ts
//...
if (obj.IsA("BasePart")) {
    obj.Touched.Connect(part =>
        part.Parent?.FindFirstChildOfClass("Humanoid")?.TakeDamage(100)
    );
}
```

Now check out what this code compiles to:

```lua
--...
if obj:IsA("BasePart") then
    obj.Touched:Connect(function(part)
        local _result = part.Parent
        if _result ~= nil then
            _result = _result:FindFirstChildOfClass("Humanoid")
            if _result ~= nil then
                _result = _result:TakeDamage(100)
            end
        end
        return _result
    end)
end
```

With our use of `?` between methods, we are telling the compiler that we aren't entirely sure that this object will be present when this code is ran. The compiler automatically wraps our code in nil-checks to ensure that it never errors. Brilliant!

Here is the completed snippet:

```ts
import { CollectionService } from "@rbxts/services";

for (const obj of CollectionService.GetTagged("Lava")) {
    if (obj.IsA("BasePart")) {
        obj.Touched.Connect(part =>
            part.Parent?.FindFirstChildOfClass("Humanoid")?.TakeDamage(100)
        );
    }
}
```

Now, let's tag a part with the `"Lava"` tag and play-test. If you touch this part, your character dies!

