Certainly! I'll provide you with an extended example that includes a callback from C++ to Java using JNI. Note that the use of callbacks requires careful consideration of threading and object lifetimes. The shared memory example will be a bit more complex, so let's start with the callback example:

Callback Example:
C++ Code (Example.cpp):

#include <iostream>

// Define a callback function pointer type
typedef void (*CallbackFunction)(const char*);

// Global variable to store the callback function
CallbackFunction gCallbackFunction = nullptr;

extern "C" {

    // Set the callback function
    void setCallbackFunction(CallbackFunction callback) {
        gCallbackFunction = callback;
    }

    // Call the callback function
    void callCallback(const char* message) {
        if (gCallbackFunction) {
            gCallbackFunction(message);
        }
    }

    void helloFromCpp() {
        std::cout << "Hello from C++!" << std::endl;

        // Call the callback function
        callCallback("Callback message from C++");
    }
}

Java Code (Example.java):

public class Example {
    static {
        System.load("/path/to/libExample.so");
    }

    // Native methods
    private native void helloFromCpp();
    private native void setCallbackFunction(CallbackFunction callback);

    // Java callback function
    public void javaCallback(String message) {
        System.out.println("Callback message from Java: " + message);
    }

    public static void main(String[] args) {
        Example example = new Example();

        // Set the Java callback function
        example.setCallbackFunction(example::javaCallback);

        // Call the C++ function
        example.helloFromCpp();
    }

    // Define the callback function pointer type
    public interface CallbackFunction {
        void callback(String message);
    }
}

JNI Implementation (ExampleJNI.cpp):

#include "Example.h"
#include <iostream>

// Global variable to store the Java callback object
jobject gCallbackObject = nullptr;

JNIEXPORT void JNICALL Java_Example_setCallbackFunction(JNIEnv *env, jobject obj, jobject callback) {
    // Save the global reference to the Java callback object
    gCallbackObject = env->NewGlobalRef(callback);
}

JNIEXPORT void JNICALL Java_Example_callCallback(JNIEnv *env, jclass clazz, jstring message) {
    // Check if the Java callback object is set
    if (gCallbackObject != nullptr) {
        // Convert the Java string to a C string
        const char* cMessage = env->GetStringUTFChars(message, nullptr);

        // Get the class of the Java callback object
        jclass callbackClass = env->GetObjectClass(gCallbackObject);

        // Get the method ID of the callback method
        jmethodID callbackMethod = env->GetMethodID(callbackClass, "callback", "(Ljava/lang/String;)V");

        // Call the Java callback method
        env->CallVoidMethod(gCallbackObject, callbackMethod, env->NewStringUTF(cMessage));

        // Release the C string
        env->ReleaseStringUTFChars(message, cMessage);
    }
}

Compile the JNI code and run the Java program as described in the previous example. This extended example demonstrates how to set a callback function in C++ from Java and invoke it from the C++ code.
