redlock-cpp
===========

STEPS TO RUN:
1. Start a redis server
    `redis server`
2. Open redis-cli to view
    `redis-cli`
3. Go to redlock-cpp-master/hiredis
    `cd $/redlock-cpp-master/hiredis`
    `make clean all`
4. Go to redlock-cpp-master
    `cd $/redlock-cpp-master`
    `make all`
5. Go to redlock-cpp-master/bin and link the hiredis
    `ln -sfn ../hiredis/libhiredis.dylib $/redlock-cpp-master/bin/libhiredis.0.11.dylib`
6. In bin run the LockExample
    `./LockExample`
7. This creates a "foo" entry in the redis cache with a unique id as value.
   Check LockExample for code.

C++ redis
redlock-cpp - Redis distributed locks in C++

Based on Redlock-rb by Salvatore Sanfilippo

This library implements the Redis-based distributed lock manager algorithm described in this blog post.

To create a lock manager:

    CRedLock * dlm = new CRedLock();
    dlm->AddServerUrl("127.0.0.1", 5005);
    dlm->AddServerUrl("127.0.0.1", 5006);
    dlm->AddServerUrl("127.0.0.1", 5007);

To acquire a lock:

    CLock my_lock;
    bool flag = dlm->Lock("my_resource_name", 1000, my_lock);
or(will alway hold the lock until release it):

    CLock my_lock;
    bool flag = dlm->ContinueLock("my_resource_name", 1000, my_lock);

Where the resource name is an unique identifier of what you are trying to lock and 1000 is the number of milliseconds for the validity time.

The returned value is false if the lock was not acquired (you may try again), otherwise an class representing the lock is returned, having three keys:

    class CLock {
    public:
        int m_validityTime; => 9897.3020019531 
        sds m_resource; => my_resource_name
        sds m_val; => 53771bfa1e775 
    };

validity, an integer representing the number of milliseconds the lock will be valid.
resource, the name of the locked resource as specified by the user.
token, a random token value which is used to safe reclaim the lock.
To release a lock:

    dlm->Unlock(my_lock);
    
It is possible to setup the number of retries (by default 3) and the retry delay (by default 200 milliseconds) used to acquire the lock.

The retry delay is actually chosen at random between $retryDelay / 2 milliseconds and the specified $retryDelay value.

Disclaimer: As stated in the original antirez's version, this code implements an algorithm which is currently a proposal, it was not formally analyzed. Make sure to understand how it works before using it in your production environments.
