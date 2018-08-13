# LockfreeSPSCBuffer
This is header only, c++ lockfree single producer single consumer buffer class for concurrent programming.
As this is header only no need for separate compilation or linking, just include the header file in your project and you can use the class.
This class design uses c++11 features so your compiler should support c++11 standard. This class is best suited for heavy data movement between 
single producer and single consumer. Typical application scenarios are video streaming pipeline. The unique design of the 
class ensures there are no additional memory copies while sharing the data between threads. 
main.cpp file shows a sample file copy program demonstrating the class usage.

# Usage
typedef LockfreeSPSCBuffer<char, 4096, BufferAllocUsingNew> LFBUFFER;
LFBUFFER sb(1024 * 1024);

As shown above this is templated class, the first parameter is the type of objects which this buffer will hold. The second and third parameters are alignment and allocator policies. keep these parameters as is, we plan to develop more cache friendly allocation policies in future. you can use a typedef as shown above just to short hand template notation.
The constructor takes total object count as argument. in the above example 1024*1024 number of char objects are created

# Writing into the buffer
    char* wptr;
		if (sb.AquireWritePtr(wptr)) {
			filein.read(wptr, 1024);
			sb.ReleaseWritePtr(filein.gcount());
		}
The method AquireWritePtr takes a (T*&) as argument and returns a bool. if there is space to write in the buffer, the function returns true or else false. then the wptr can be given to the file object to read data into it. Once the ptr is aquired it has to be released using the call ReleaseWritePtr(), this takes in number of objects written into the buffer.

# Reading from the buffer
  	std::pair<char*, int> rinfo;
		if (sb.AquireReadPtr(rinfo)) {
			fout.write(rinfo.first, rinfo.second);
			sb.ReleaseReadPtr(rinfo.second);
    }
  Reading is similar to writing, first aquire the read ptr and then release the readptr. There are couple of differences though, first is AquireReadPtr takes reference to an std::pair<T*,int> as its argument. this will give the readptr and the readsize. 
  
# Communicating end of stream
  producer can notify consumer about the end of stream condition. The following are the functions for the same
   SetEOS(), GetEOS()
   These are thread safe.
  
# Resetting buffer
   In some scenarios the pipeline might require a reset. To do that you can use
   ResetCircularBuffer(). But note that this call is not thread safe
  
  Complete code example can be found in main.cpp. 


