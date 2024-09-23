# rohit72
Asse
#include <stdio.h>
#include <stdint.h>

#define HEAP_SIZE 128  // Heap size is 128KB, with each block being 1KB
#define BLOCK_SIZE 1   // 1KB block size
#define MIN_BLOCK_SIZE BLOCK_SIZE

typedef struct Block {
    size_t size;         // Size of the block in KB
    int free;            // Whether the block
    struct Block* next;  // Pointer to next
} Block;

static uint8_t heap[HEAP_SIZE * 1024];  // heap 
static Block* freeList = (Block*)heap;  // Free list starts of the heap

// Initialize the heap 
void initHeap() {
    freeList->size = HEAP_SIZE;  // 128KB free 
    freeList->free = 1;          // free
    freeList->next = NULL;       
}

// Find the block
Block* findBestFit(size_t size) {
    Block* current = freeList;
    Block* bestFit = NULL;

    while (current != NULL) {
        if (current->free && current->size >= size) {
            if (bestFit == NULL || current->size < bestFit->size) {
                bestFit = current;
            }
        }
        current = current->next;
    }

    return bestFit;
}

// Malloc fun
void* malloc(size_t size) {
    if (size == 0 || size > HEAP_SIZE * 1024) {
        return NULL;  // Invalid size
    }

    // Round 1KB
    size_t numBlocks = (size + 1023) / 1024;

    // Find blcok
    Block* bestFit = findBestFit(numBlocks);
    if (!bestFit) {
        return NULL;  // if no block found
    }

    // Allocate memory 
    if (bestFit->size > numBlocks) {
        Block* newBlock = (Block*)((uint8_t*)bestFit + numBlocks * 1024);
        newBlock->size = bestFit->size - numBlocks;
        newBlock->free = 1;
        newBlock->next = bestFit->next;
        bestFit->next = newBlock;
    }

    bestFit->size = numBlocks;
    bestFit->free = 0;  // using block
    return (void*)((uint8_t*)bestFit + sizeof(Block));
}


void free(void* ptr) {
    if (ptr == NULL) {
        return;
    }

    // Get the block
    Block* block = (Block*)((uint8_t*)ptr - sizeof(Block));
    block->free = 1;  //  free

    // Merge free blocks
    Block* current = freeList;
    while (current != NULL) {
        if (current->free && current->next && current->next->free) {
            // Mergeing current block with next
            current->size += current->next->size;
            current->next = current->next->next;
        }
        current = current->next;
    }
}

int main() {
    initHeap();  

    
    void* ptr1 = malloc(sizeof(int) * 128);  // 1 Kb block
    void* ptr2 = malloc(sizeof(uint8_t) * 1000);  //  1 kb block
    void* ptr3 = malloc(128 * 8 * 1024);  // 128kb  block

    
    free(ptr1);
    free(ptr2);
    free(ptr3);

    return 0;
}


