cmake_minimum_required(VERSION 2.6)
project(libevent C)
set(_EVENT_VERSION "2.0.21-stable")
set(_EVENT_NUMERIC_VERSION 0x02001500)

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(include)
include_directories(compat)

# Configuration
find_package(OpenSSL)
set(_EVENT_HAVE_OPENSSL OpenSSL_FOUND)

find_package(zlib)
set(_EVENT_HAVE_LIBZ ZLIB_FOUND)

set(CMAKE_THREAD_PREFER_PTHREAD yes)
find_package(Threads)
set(_EVENT_HAVE_PTHREADS ${CMAKE_USE_PTHREADS_INIT})

include(platform_checks.cmake)
configure_file(event-config.h.in include/event2/event-config.h)
configure_file(event-config.h.in event2/event-config.h)

# Sources

set(SYS_SRC)
set(NEED_SIGNAL)
# event_ports
if(_EVENT_HAVE_EVENT_PORTS)
	list(APPEND SYS_SRC "evport.c")
	set(NEED_SIGNAL yes)
endif(_EVENT_HAVE_EVENT_PORTS)
# poll
if(_EVENT_HAVE_POLL)
	list(APPEND SYS_SRC "poll.c")
	set(NEED_SIGNAL yes)
endif(_EVENT_HAVE_POLL)
# select
if(_EVENT_HAVE_SELECT)
	list(APPEND SYS_SRC "select.c")
	set(NEED_SIGNAL yes)
endif(_EVENT_HAVE_SELECT)
# kqueue
if(_EVENT_HAVE_WORKING_KQUEUE)
	list(APPEND SYS_SRC "kqueue.c")
endif(_EVENT_HAVE_WORKING_KQUEUE)
# devpoll
if(_EVENT_HAVE_DEVPOLL)
	list(APPEND SYS_SRC "devpoll.c")
endif(_EVENT_HAVE_DEVPOLL)
# epoll
if(_EVENT_HAVE_EPOLL)
	list(APPEND SYS_SRC "epoll.c")
	set(NEED_SIGNAL yes)
endif(_EVENT_HAVE_EPOLL)
# signal
if(NEED_SIGNAL)
	list(APPEND SYS_SRC signal.c)
endif(NEED_SIGNAL)

set(CORE_SRC
	event.c evthread.c buffer.c
	bufferevent.c bufferevent_sock.c bufferevent_filter.c
	bufferevent_pair.c listener.c bufferevent_ratelim.c
	evmap.c log.c evutil.c evutil_rand.c strlcpy.c ${SYS_SRC})

set(EXTRA_SRC
	event_tagging.c http.c evdns.c evrpc.c)

# Full library
add_library(event SHARED
	${CORE_SRC}
	${EXTRA_SRC})

# Core
add_library(event_core SHARED
	${CORE_SRC})

# Pthreads
add_library(event_pthreads SHARED
	evthread_pthread.c)
target_link_libraries(event_pthreads event_core)

# Extra
add_library(event_extra SHARED
	${EXTRA_SRC})
target_link_libraries(event_extra event_core)

# OpenSSL
if(OpenSSL_FOUND)
add_library(event_openssl SHARED
	bufferevent_openssl.c)
target_link_libraries(event_openssl
	event_core
	${OPENSSL_LIBRARIES})
endif(OpenSSL_FOUND)
