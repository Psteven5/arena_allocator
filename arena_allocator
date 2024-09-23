/*
BSD 3-Clause License

Copyright (c) 2024, Pieter Stevens

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its
   contributors may be used to endorse or promote products derived from
   this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
*/

#ifndef ARENA_ALLOCATOR_INCLUDED
#define ARENA_ALLOCATOR_INCLUDED

#include <iostream>
#include <memory>

#if 201100 <= __cplusplus

template<typename _Tp>
struct arena_allocator {
    // size of an allocation page of the underlying memory
    constexpr static
    std::size_t PAGE_SIZE = 8192;

    using value_type = _Tp;

    constexpr
    arena_allocator() noexcept
        : m(std::make_shared<struct_arena_allocator__struct_m>()) {}

    // allocates n of type _Tp and returns a pointer to the memory
    #if 201400 <= __cplusplus
        constexpr
    #endif
    auto allocate(std::size_t n) -> _Tp *
    {
        if (0 == n)
            throw std::bad_alloc{};
        reserve(n);
        const auto ptr = m->curr;
        m->curr += n;
        return ptr;
    }

    // deallocates if it is the most recent allocation, otherwise does nothing
    constexpr
    void deallocate(_Tp *ptr, std::size_t n) noexcept {
        if (ptr == m->curr - n)
            m->curr = ptr;
    }

    // allocates the necessary memory so allocating n of type _Tp does not need further allocations
    #if 201400 <= __cplusplus
        constexpr
    #endif
    void reserve(std::size_t n)
    {
        if (m->curr + n > (_Tp *)(m->page + 1) + m->page->size) {
            auto &next = m->page->next;

            if (nullptr == next) {
                const auto mn = std::max(n, PAGE_SIZE / sizeof(_Tp));
                if (!(next = (struct_arena_allocator__struct_page *)malloc(mn * sizeof(_Tp) + sizeof(*next))))
                    throw std::bad_alloc{};
                *next = {mn, nullptr};
            } else if (n * sizeof(_Tp) > next->size) {
                if (!(next = (struct_arena_allocator__struct_page *)realloc(next, n * sizeof(_Tp) + sizeof(*next))))
                    throw std::bad_alloc{};
                next->size = n;
            }

            m->page = next;
            m->curr = (_Tp *)(next + 1);
        }
    }

    // resets the arena without deallocating any allocation, allowing for reuse
    constexpr
    void clear() noexcept
    {
        m->page = &m->pages;
        m->curr = (_Tp *)(&m->pages + 1);
    }

    // deallocates all allocations in the arena, also called by destructor
    constexpr
    void destroy() noexcept
    {
        m->destroy();
        clear();
    }

    private:

    struct struct_arena_allocator__struct_page {
        std::size_t size;
        struct_arena_allocator__struct_page *next;
    };

    struct struct_arena_allocator__struct_m {
        struct_arena_allocator__struct_page pages = {0, nullptr}, *page = &pages;
        _Tp *curr = (_Tp *)(&pages + 1);

        #if 201700 <= __cplusplus
            constexpr
        #endif
        void destroy() noexcept
        {
            for (auto page = pages.next; nullptr != page;) {
                const auto next = page->next;
                free(page);
                page = next;
            }
            pages.next = nullptr;
        }

        #if 202000 <= __cplusplus
            constexpr
        #endif
        ~struct_arena_allocator__struct_m() noexcept
        {
            destroy();
        }
    };

    const std::shared_ptr<struct_arena_allocator__struct_m> m;
};

#else

#pragma message "arena_allocator expects C++11 or higher"
#error 201100 > __cplusplus

#endif // 201100 <= __cplusplus

#endif // ARENA_ALLOCATOR_INCLUDED
