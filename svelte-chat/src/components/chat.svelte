<script>
  import { onDestroy, onMount, afterUpdate } from "svelte";
  import pb, { currentUser } from "../lib/pocketbaseInit";
  import { navigate } from "svelte-routing";

  let messagesData = [];
  $: messagelikes = [];

 
  let unsubscribe;
  let unsubscribeLikes;
  let container;
  let shouldScroll = true;
  let likeCounts = [];
  onMount(async () => {

    if (!$currentUser) {
      navigate("/login", { replace: true });
    } else {
      let data = await pb.collection("messages").getList(1, 50, {
        sort: "created",
        expand: `field`,
      });
      let likeData = await pb.collection("messageLikes").getList(1, 50, {
        sort: "created",
        expand: "post",
      });

      messagelikes = likeData.items;

      messagesData = data.items;

      unsubscribeLikes = await pb
        .collection("messageLikes")
        .subscribe("*", async ({ action, record }) => {
          if (action === "update") {
            userLikeInner.forEach((item) => {
              if (item.messageId === record.post) {
                item.like = record.like;
                item.love = record.love;
                item.fire = record.fire;
                item.dump = record.dump;
              }
            });

            messagelikes.forEach((item) => {
              if (item.post === record.post) {
                item.like = record.like;
                item.love = record.love;
                item.fire = record.fire;
                item.dump = record.dump;
                item.post = record.post;
                item.userLike = record.userLike;
              }
            });

         
          

            // record.expand = { field };
            // userLikeInner.forEach((item) => {
            //   field.forEach((itemFiled) => {
            //     if (item.messageId === itemFiled.collectionId) {
            //       console.log(item);
            //     }
            //   });
            // });

            // messagesData = [...messagesData, record];
          }
        });

      // get like counts

      //   subscriibe to realtime messages
      unsubscribe = await pb
        .collection("messages")
        .subscribe("*", async ({ action, record }) => {
          if (action === "create") {
            const field = await pb.collection("users").getOne(record.field);
            record.expand = { field };

            messagesData = [...messagesData, record];
          }
          if (action === "delete") {
            messagesData = messagesData.filter((m) => m.id !== record.id);
          }
        });
    }
  });

  afterUpdate(() => {
    console.log("afterUpdate");
    if (messagesData) scrollToBottom(container);
  });
  const scrollToBottom = async (node) => {
    node.scroll({ top: node.scrollHeight, behavior: "smooth" });
  };

  onDestroy(() => {
    unsubscribe?.();
    unsubscribeLikes?.();
  });

  //   handlle signout
  const handleSignout = () => {
    pb.authStore.clear();
    navigate("/login", { replace: true });
  };
  function countLikes(postId, property) {
    let count = 0;
    for (let like of messagelikes) {
      if (like.post === postId && like[property]) {
        count++;
      }
    }
    return count;
  }
  //   handle message Sumbit
  let messages = {};
  $: disabled = false;

  $: if (messages.text === "" || messages.text === undefined) {
    disabled = true;
  } else {
    disabled = false;
  }
  const handleMessageSubmit = async () => {
    try {
      let resposneMessage = await pb.collection("messages").create({
        text: messages.text,
        field: $currentUser.id,
      });

      console.log(resposneMessage);
    } catch (error) {
      console.log(error);
    }
  };
  let userLikeInner = [];

  const fetchData = async () => {
    const storedData = localStorage.getItem("userLikeInner");
    if (storedData) {
      userLikeInner = JSON.parse(storedData);
    } else {
      // Retrieve existing like data from the backend and initialize userLikeInner array
      const retriveDataProfile = await pb
        .collection("messageLikes")
        .getFullList({
          sort: "created",
        });

      retriveDataProfile.forEach((users) => {
        const existingEntry = {
          id: users.post,
          like: users.like,
          love: users.love,
          fire: users.fire,
          dump: users.dump,
        };
        userLikeInner.push(existingEntry);
      });

      localStorage.setItem("userLikeInner", JSON.stringify(userLikeInner));
    }
  };

  fetchData();

  const handleLike = async (button, post, messageId) => {

  
    const existingIndex = userLikeInner.findIndex((item) => item.id === post);

    if (existingIndex !== -1) {
      // Update existing entry
      userLikeInner[existingIndex][button] =
        !userLikeInner[existingIndex][button];
      console.log(userLikeInner, userLikeInner[existingIndex][button]);
    } else {
      // Create new entry
      const newEntry = {
        id: post,
        messageId: messageId,
        [button]: true,
      };
      userLikeInner.push(newEntry);
    }

    for (const item of userLikeInner) {
      if (item.messageId === messageId) {
        const retriveDataProfile = await pb
          .collection("messageLikes")
          .getFullList({
            sort: "created",
          });

        const userExists = retriveDataProfile.some(
          (users) =>
            users.userLike.includes(pb.authStore.model.id) &&
            users.post === item.messageId
        );

        if (userExists) {
          const likeEntry = retriveDataProfile.find(
            (users) =>
              users.userLike.includes(pb.authStore.model.id) &&
              users.post === item.messageId
          );

          await pb.collection("messageLikes").update(likeEntry.id, {
            like: item.like,
            love: item.love,
            fire: item.fire,
            dump: item.dump,
            post: item.messageId,
            userLike: pb.authStore.model.id,
          });
        } else {
          await pb.collection("messageLikes").create({
            like: item.like || false,
            love: item.love || false,
            fire: item.fire || false,
            dump: item.dump || false,
            post: item.messageId,
            userLike: pb.authStore.model.id,
          });
        }
      }
    }

    localStorage.setItem("userLikeInner", JSON.stringify(userLikeInner));
  };

  // // handle like functionality
  // const handleLike = async (button, post, messageId) => {
  //   const existingIndex = userLikeInner.findIndex((item) => item.id === post);
  //   if (existingIndex !== -1) {
  //     // Update existing entry
  //     userLikeInner[existingIndex][button] =
  //       !userLikeInner[existingIndex][button];
  //   } else {
  //     // Create new entry
  //     const newEntry = { id: post, messageId: messageId };
  //     newEntry[button] = true;
  //     userLikeInner.push(newEntry);
  //   }

  //   userLikeInner.forEach(async (item) => {
  //     if (item.messageId === messageId) {
  //       const retriveDataProfile = await pb
  //         .collection("messageLikes")
  //         .getFullList({
  //           sort: "created",
  //         });

  //       const userExists = retriveDataProfile.some((users) => {
  //         item.likeId = users.id;
  //         if (
  //           users.userLike.includes(pb.authStore.model.id) &&
  //           users.post === item.messageId
  //         ) {

  //         }

  //         return (
  //           users.userLike.includes(pb.authStore.model.id) &&
  //           users.post === item.messageId
  //         );
  //       });

  //       if (userExists) {
  //         const likeResponse = await pb
  //           .collection("messageLikes")
  //           .update(item.likeId, {
  //             like: item.like,
  //             love: item.love,
  //             fire: item.fire,
  //             dump: item.dump,
  //             post: item.messageId,
  //             userLike: pb.authStore.model.id,
  //           });
  //       } else {
  //         const likeResponse = await pb.collection("messageLikes").create({
  //           like: item.like || false,
  //           love: item.love || false,
  //           fire: item.fire || false,
  //           dump: item.dump || false,
  //           post: item.messageId,
  //           userLike: pb.authStore.model.id,
  //         });
  //       }
  //     }
  //   });
  // };
</script>

{#if $currentUser}
  <div class="container chat-main">
    <div class="grid profile">
      <h6>
        Hi @{$currentUser.username} <small>({$currentUser.id}) 🔥🔥</small>
      </h6>
      <div class="grid">
        <button class="btn secondary" on:click={handleSignout}>Sign out</button>
      </div>
    </div>

    <div bind:this={container} class="message-list">
      <div class="grid message-lists">
        {#if messagesData === null}
          <div>No data found</div>
        {:else}
          {#await messagesData}
            <li><span>Loading...</span></li>
          {:then messagesData}
            {#each messagesData as message, index}
              <div class="chat-list">
                <div class="chat-list-user">
                  <div class="avatar">
                    <img src="../src/assets/avatar.jpg" alt="" />
                    <div class="title">
                      <small>Sent by @{message.expand?.field?.username}</small>
                      <span class="messages">{message.text}</span>
                    </div>
                  </div>

                  {#if messagelikes}
                    <div class="post-likes container">
                      <div class="post-likes-wrapper">
                        <div class="grid post-like">
                          <div class="like-wrapper">
                            <button
                              class="likes"
                              on:click={() =>
                                handleLike("like", index, message.id)}
                              >👌</button
                            >

                            <span>{countLikes(message.id, "like")}</span>
                          </div>
                          <div class="like-wrapper">
                            <button
                              on:click={() =>
                                handleLike("love", index, message.id)}
                              class="likes">💓</button
                            >
                            <span>{countLikes(message.id, "love")}</span>
                          </div>
                          <div class="like-wrapper">
                            <button
                              on:click={() =>
                                handleLike("fire", index, message.id)}
                              class="likes">🔥</button
                            >
                            <span>{countLikes(message.id, "fire")}</span>
                          </div>

                          <div class="like-wrapper">
                            <button
                              on:click={() =>
                                handleLike("dump", index, message.id)}
                              class="likes">💩</button
                            >
                            <span>{countLikes(message.id, "dump")}</span>
                          </div>
                        </div>
                      </div>
                    </div>
                  {/if}
                </div>
              </div>
            {/each}
          {/await}
        {/if}
      </div>
    </div>
  </div>

  <div class="container chat-data">
    <div class="grid">
      <div class="chatinput">
        <form on:submit|preventDefault>
          <input
            bind:value={messages.text}
            type="text"
            placeholder="type your messages"
          />
          <button {disabled} on:click={handleMessageSubmit}>Send</button>
        </form>
      </div>
    </div>
  </div>
{/if}

<style>
  .post-like {
    display: flex;
  }
  .like-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .like-wrapper span {
    margin-top: -5px;
    font-size: 0.8rem;
  }
  .likes {
    background-color: #172129;
  }
  .chat-list-user {
    display: flex;
    flex-direction: column;
    justify-content: left;
  }
  .post-likes {
    display: flex;
  }
  .post-likes-wrapper {
    width: 60%;
    display: flex;
    gap: 12px;
    padding-top: 10px;
    padding-bottom: 10px;
  }
  .message-lists {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }
  .message-list {
    background-color: #172129;

    padding: 20px;
    height: 63vh;
    overflow-y: scroll;
  }

  .chat-list {
    background-color: #182c3c;
    padding: 20px;
  }
  .chat-list .avatar {
    display: flex;
    flex-direction: row;
    gap: 12px;
  }
  .chat-list .avatar .title {
    display: flex;
    flex-direction: column;
  }
  .chat-list .avatar .title span {
    font-size: 1.2em;
    font-weight: 500;
    color: #2485d5;
  }
  .chat-list .avatar img {
    width: 50px;
    height: 50px;
  }
  .chat-main {
    display: flex;
    justify-content: center;
    padding-top: 25px;
    flex-direction: column;

    width: 100%;
  }
  .container h6 {
    font-size: 0.8rem;
    font-weight: 400;
  }

  .profile {
    display: flex;
    flex-direction: column;
  }
  .profile .btn {
    padding: 10px;
    font-size: 1em;
    font-weight: 600;
  }

  .chat-data {
    position: fixed;
    width: 100%;
    bottom: 0px;
  }

  .chatinput form {
    display: flex;
    flex-direction: row;
    gap: 20px;
  }
  .chatinput form button {
    flex: 1;
  }
  .chatinput form input {
    flex: 5;
  }
</style>
