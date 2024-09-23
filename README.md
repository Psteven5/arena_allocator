# arena_allocator
C++11 single header arena allocator that works with the STL.

Keep in mind that because the C++ committee has decided to reject a reallocation trait for allocators, repeated reallocations - for example vector::push_back() in a loop - will waste a lot of memory space (as deallocation is done after allocation), so be sure to reserve when possible!

The only memory-reuse optimization that works for now is deallocation of the most recent allocation, which is a simple pointer decrement. This implementation also drops all allocated memory in the destructor, with the ability of also doing this explicitly through arena_allocator::destroy(). As a bonus, there is also arena_allocator::clear(), which keeps the pages allocated, allowing for reuse.

(I will push an update whenever something akin to a reallocation trait ever comes. However, a larger part of the standard library than just std::allocator would have to be reimplemented, so don't count on it)
