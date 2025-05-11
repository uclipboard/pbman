#!/usr/bin/env osascript -l JavaScript
"use strict";

const APP_NAME = "pbman";
const DELAY_TIME = 0.4; // seconds

const app = Application.currentApplication();
app.includeStandardAdditions = true;
// Import the AppKit framework to access NSPasteboard
ObjC.import("AppKit");

const stdin = $.NSFileHandle.fileHandleWithStandardInput;
const stdout = $.NSFileHandle.fileHandleWithStandardOutput;
const stderr = $.NSFileHandle.fileHandleWithStandardError;

function eprint(msg) {
    // Print to stderr
    const data = $.NSString.alloc.initWithUTF8String(msg);
    stderr.writeData(data.dataUsingEncoding($.NSUTF8StringEncoding));
}

function eprintln(msg) {
    eprint(msg + "\n");
}

function execute(exec, args) {
    // execute the command
    const task = $.NSTask.alloc.init;
    task.launchPath = exec;
    task.standardOutput = stdout;
    task.standardError = stderr;
    task.arguments = args;

    // to fix the EOF problem
    const inputPipe = $.NSPipe.pipe;
    task.standardInput = inputPipe;

    task.launch;
    const pipWriter = inputPipe.fileHandleForWriting;
    for (let chunk = stdin.availableData; chunk.length > 0; chunk = stdin.availableData) {
        pipWriter.writeData(chunk);
    }
    pipWriter.closeFile;

    task.waitUntilExit;
    const status = task.terminationStatus;
    if (status !== 0) {
        eprintln("Error: Command failed with status " + status);
        return;
    }
}

function PasteBoardChangeWatchLoop(F) {
    // Get the general pasteboard
    const pasteboard = $.NSPasteboard.generalPasteboard;

    let changeCount = pasteboard.changeCount;

    while (true) {
        if (changeCount !== pasteboard.changeCount) {
            // eprintln("Change detected in pasteboard.");
            try {
                F();
            } catch (e) {
                eprintln("Error occurred in PasteBoardChangeWatchLoop: " + e);
            }
        }
        changeCount = pasteboard.changeCount;
        delay(DELAY_TIME)
    }
}

function copy(args) {
    // execute the pbcopy command
    execute("/usr/bin/pbcopy", args);
}

function paste(args) {
    // execute the pbpaste command
    execute("/usr/bin/pbpaste", args);
}

function watch(cmd) {
    if (cmd.length === 0) {
        eprintln("No command provided to watch.");
        return;
    }

    // join the command arguments into a single string
    cmd = cmd.join(" ");
    PasteBoardChangeWatchLoop(() => {
        // this command would be a shell command after join
        // so let's execute the command with sh
        execute("/bin/sh", ["-c", cmd]);
    });
}

function help() {
    eprintln(`${APP_NAME}(https://github.com/uclipboard/pbman) - A simple pasteboard manager wrapper for macOS, a part of uclipboard.`);
    eprintln(`Usage: ${APP_NAME} <sub-command> <options>...`);
    eprintln("Commands:");
    eprintln("  watch <command> Watch the clipboard and execute the command when it changes, " +
        "e.g. `watch 'echo Hello World'` or `watch echo Hello World.`, it will concatenate the command so don't worry.");
    eprintln("  copy <options>\t\tCopy text from stdin to the clipboard by pbcopy, <options> will be passed to pbcopy.");
    eprintln("  paste <options>\t\tPaste text from the clipboard by pbpaste, <options> will be passed to pbpaste.");
}

function run(argv) {
    if (!argv[0]) {
        help();
        return;
    }
    switch (argv[0]) {
        case "watch":
            watch(argv.slice(1));
            break;
        case "copy":
            copy(argv.slice(1));
            break;
        case "paste":
            paste(argv.slice(1));
            break;
        default:
            help();
            break;
    }
}

