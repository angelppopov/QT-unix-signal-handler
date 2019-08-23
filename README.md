# QT-unix-signal-handler


Unix signals in Qt applications
It's quite easy to catch unix signals in Qt applications. you may like to ignore or accept them.

#include <QCoreApplication>

#include <initializer_list>
#include <signal.h>
#include <unistd.h>

void ignoreUnixSignals(std::initializer_list<int> ignoreSignals) {
    // all these signals will be ignored.
    for (int sig : ignoreSignals)
        signal(sig, SIG_IGN);
}

void catchUnixSignals(std::initializer_list<int> quitSignals) {
    auto handler = [](int sig) -> void {
        // blocking and not aysnc-signal-safe func are valid
        printf("\nquit the application by signal(%d).\n", sig);
        QCoreApplication::quit();
    };
    
    sigset_t blocking_mask;   
    sigemptyset(&blocking_mask);  
    for (auto sig : quitSignals) 
        sigaddset(&blocking_mask, sig);  
        
    struct sigaction sa;   
    sa.sa_handler = handler;   
    sa.sa_mask    = blocking_mask;  
    sa.sa_flags   = 0;    
    
    for (auto sig : quitSignals)   
        sigaction(sig, &sa, nullptr);
}

int main(int argc, char *argv[]) {
    QCoreApplication app(argc, argv);
    catchUnixSignals({SIGQUIT, SIGINT, SIGTERM, SIGHUP});

    // do your initialization and other stuff
    
    
    return app.exec();
}
done!

this solution has been tested under OS X (10.9~10.11), MacOS (10.12) and Ubuntu (12.04/14.4/16.04).

note:

c++11/14 makes this code short and readable, although you can simply implement this solution under older c++ versions.

signals
by accepting signals like SIGINT (ctrl c) you can control the application shutdown process to free your resources, generate logs, ...

by shell
the following list show the most important signals a shell (bash, ...) may send to your application:

SIGINT (ctrl c): by default, this causes the process to terminate.

SIGQUIT (ctrl \): by default, this causes the process to terminate and dump core.

SIGTSTP (ctrl z): by default, this causes the process to suspend execution.

quit signals
optionally you may like to catch these signals and manually quit your application:

SIGINT : The SIGINT signal is sent to a process by its controlling terminal when a user wishes to interrupt the process. This is typically initiated by pressing Control-C, but on some systems, the "delete" character or "break" key can be used.

SIGHUP : The SIGHUP signal is sent to a process when its controlling terminal is closed. It was originally designed to notify the process of a serial line drop (a hangup). In modern systems, this signal usually means that the controlling pseudo or virtual terminal has been closed. Many daemons will reload their configuration files and reopen their logfiles instead of exiting when receiving this signal. nohup is a command to make a command ignore the signal.

SIGTERM : The SIGTERM signal is sent to a process to request its termination. Unlike the SIGKILL signal, it can be caught and interpreted or ignored by the process. This allows the process to perform nice termination releasing resources and saving state if appropriate. It should be noted that SIGINT is nearly identical to SIGTERM.

SIGQUIT: The SIGQUIT signal is sent to a process by its controlling terminal when the user requests that the process quit and perform a core dump.

dameon
you may like to ignore these signals, if your application is a daemon (daemon(0, 0);):

SIGCHLD : The SIGCHLD signal is sent to a process when a child process terminates, is interrupted, or resumes after being interrupted. One common usage of the signal is to instruct the operating system to clean up the resources used by a child process after its termination without an explicit call to the wait system call.

SIGTSTP : The SIGTSTP signal is sent to a process by its controlling terminal to request it to stop temporarily. It is commonly initiated by the user pressing Control-Z. Unlike SIGSTOP, the process can register a signal handler for or ignore the signal.

forbidden signals
you can not catch nor ignore these two signals:

SIGKILL : The SIGKILL signal is sent to a process to cause it to terminate immediately (kill). In contrast to SIGTERM and SIGINT, this signal cannot be caught or ignored, and the receiving process cannot perform any clean-up upon receiving this signal.

SIGSTOP : The SIGSTOP signal instructs the operating system to stop a process for later resumption.

other solutions
there are some other solution for catching a signal in Qt application.

for instance, QSocketNotifier can be used to emit a Qt signal when a unix signal is triggered. this link shows a how-to which is both safe and more flexible.

the solution in this gist is an easy solution mostly for closing a Qt application.

reference
Blocking Signals for a Handler

Unix signals on wikipedia
