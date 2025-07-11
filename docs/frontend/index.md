# __:material-language-html5: :material-language-css3: :material-language-typescript:__
<!-- Frontend fundamentals__ -->

## Typescript

### Types and Interfaces

- [Type v/s Interface](https://blog.logrocket.com/types-vs-interfaces-in-typescript/)

- Infer types from variables like so. This is beneficial when the type of the variable changes, we don't have to update the the type of that variable manually

    ```{.typescript .copy}
    export type VariableType = ReturnType<typeof variable>
    ```


## CSS

- [Flex](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Inset](https://developer.mozilla.org/en-US/docs/Web/CSS/inset)
- [Justify v/s Align](https://css-tricks.com/a-quick-way-to-remember-difference-between-justify-content-align-items/)
    - [justify-self](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-self): This property sets the way a box is justified inside its alignment container along the appropriate axis.
    - [justify-items](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-items#try_it): This property defines the default justify-self for all items of the box, giving them all a default way of justifying each box along the appropriate axis.
    - [justify-content](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content): This property defines how the browser distributes space between and around content items along the main-axis of a flex container, and the inline axis of a grid container.
