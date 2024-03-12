+++
title = "Data Driven Tycoons"
+++

This page is currently just for documenting my data driven tycoon system.<br>

<style>
    li::marker {
        color: #FFFFFF
    }
</style>

## Core
The primary data format, every other format inherits Core in some way shape or form.

- [dict]: The table everything in the datafile is stored in.
    - [string] **format**: A simple guiding value thats required in every datafile. Tells the data generator what the datafile is used for.
    - [dict?] **custom**: If a datafile gets a successful trigger, it will check if this table exists and excute the function + metadata within.
        - [() -> nil] **func**: This function is called when the datafile gets a trigger, in Core, this doesn't have any parameters or return values.

