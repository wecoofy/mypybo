<script>
    // @ts-nocheck

    import apicall from "../lib/api";
    import Error from "../components/Error.svelte";
    import { push, link } from "svelte-spa-router";
    import { is_login, username, answer_order } from "../lib/store";
    import { marked } from "marked";

    import moment from "moment/min/moment-with-locales";
    moment.locale("ko");

    export let params = {};
    let question_id = params.q_id;
    // console.log("question_id:" + question_id);
    let question = { answers: [], voter: [], content: "" };
    let content = "";
    let error = { detail: [] };

    function get_question() {
        apicall("get", "/api/question/detail/" + question_id, {}, (json) => {
            question = json;
        });
    }

    get_question();

    $: if ($answer_order && question.answers.length) {
        if ($answer_order === "date") {
            question.answers.sort((a, b) => new Date(a.create_date) - new Date(b.create_date));
            console.log("order: date");
        } else if ($answer_order === "vote") {
            question.answers.sort((a, b) => b.voter.length - a.voter.length);
            console.log("order: vote");
        }
        question.answers = [...question.answers];
    }

    function post_answer(event) {
        event.preventDefault();
        let url = "/api/answer/create/" + question_id;
        let params = {
            content: content,
        };
        apicall(
            "post",
            url,
            params,
            (json) => {
                content = "";
                error = { detail: [] };

                get_question();
            },
            (err_json) => {
                error = err_json;
            },
        );
    }

    function delete_question(_question_id) {
        if (window.confirm("정말로 삭제하시겠습니까?")) {
            let url = "/api/question/delete";
            let params = {
                question_id: _question_id,
            };
            apicall(
                "delete",
                url,
                params,
                (json) => {
                    push("/");
                },
                (err_json) => {
                    error = err_json;
                },
            );
        }
    }

    function delete_answer(answer_id) {
        if (window.confirm("정말로 삭제하시겠습니까?")) {
            let url = "/api/answer/delete";
            let params = {
                answer_id: answer_id,
            };
            apicall(
                "delete",
                url,
                params,
                (json) => {
                    get_question();
                },
                (err_json) => {
                    error = err_json;
                },
            );
        }
    }

    function vote_question(_question_id) {
        if (window.confirm("정말로 추천하시겠습니까?")) {
            let url = "/api/question/vote";
            let params = {
                question_id: _question_id,
            };
            apicall(
                "post",
                url,
                params,
                (json) => {
                    get_question();
                },
                (err_json) => {
                    error = err_json;
                },
            );
        }
    }

    function vote_answer(answer_id) {
        if (window.confirm("정말로 추천하시겠습니까?")) {
            let url = "/api/answer/vote";
            let params = {
                answer_id: answer_id,
            };
            apicall(
                "post",
                url,
                params,
                (json) => {
                    get_question();
                },
                (err_json) => {
                    error = err_json;
                },
            );
        }
    }
</script>

<div class="container my-3">
    <!-- 질문 -->
    <h2 class="border-bottom py-2">{question.subject}</h2>
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text">
                {@html marked.parse(question.content)}
            </div>
            <div class="d-flex justify-content-end">
                {#if question.modify_date}
                    <div class="badge bg-light text-dark p-2 text-start mx-3">
                        <div class="mb-2">수정된 날짜</div>
                        <div>
                            {moment(question.modify_date).format("YYYY년 MM월 DD일 HH:mm")}
                        </div>
                    </div>
                {/if}
                <div class="badge bg-light text-dark p-2 text-start">
                    <div class="mb-2">
                        {question.user ? question.user.username : ""}
                    </div>
                    <div>
                        {moment(question.create_date).format("YYYY년 MM월 DD일 HH:mm")}
                    </div>
                </div>
            </div>
            <div class="my-3">
                <button class="btn btn-sm btn-outline-secondary" on:click={vote_question(question.id)}>
                    추천
                    <span class="badge rounded-pill bg-success">{question.voter.length}</span>
                </button>
                {#if question.user && $username === question.user.username}
                    <a use:link href="/question-modify/{question.id}" class="btn btn-sm btn-outline-secondary">수정</a>
                    <button class="btn btn-sm btn-outline-secondary" on:click={() => delete_question(question.id)}>삭제</button>
                {/if}
            </div>
        </div>
    </div>
    <!-- 답변 목록 -->
    <div class="d-flex justify-content-between">
        <h5 class="border-bottom my-3 py-2">
            {question.answers.length}개의 답변이 있습니다.
        </h5>
        <select class="form-select" style="width: 10ch" bind:value={$answer_order}>
            <option value="date">날짜순</option>
            <option value="vote">추천순</option>
        </select>
    </div>
    {#each question.answers as answer}
        <div class="card my-3">
            <div class="card-body">
                <div class="card-text">
                    {@html marked.parse(answer.content)}
                </div>
                <div class="d-flex justify-content-end">
                    {#if answer.modify_date}
                        <div class="badge bg-light text-dark p-2 text-start mx-3">
                            <div class="mb-2">수정된 날짜</div>
                            <div>
                                {moment(answer.modify_date).format("YYYY년 MM월 DD일 HH:mm")}
                            </div>
                        </div>
                    {/if}
                    <div class="badge bg-light text-dark p-2 text-start">
                        <div class="mb-2">
                            {answer.user ? answer.user.username : ""}
                        </div>
                        <div>
                            {moment(answer.create_date).format("YYYY년 MM월 DD일 HH:mm")}
                        </div>
                    </div>
                </div>
                <div class="my-3">
                    <button class="btn btn-sm btn-outline-secondary" on:click={vote_answer(answer.id)}>
                        추천
                        <span class="badge rounded-pill bg-success">{answer.voter.length}</span>
                    </button>
                    {#if answer.user && $username === answer.user.username}
                        <a use:link href="/answer-modify/{answer.id}" class="btn btn-sm btn-outline-secondary">수정</a>
                        <button class="btn btn-sm btn-outline-secondary" on:click={() => delete_answer(answer.id)}>삭제</button>
                    {/if}
                </div>
            </div>
        </div>
    {/each}
    <!-- 답변 등록 -->
    <Error {error} />
    <form method="post" class="my-3">
        <div class="mb-3">
            <textarea rows="10" bind:value={content} disabled={!$is_login} class="form-control"></textarea>
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary" disabled={!$is_login} on:click={post_answer} />
    </form>
    <button
        class="btn btn-secondary"
        on:click={() => {
            push("/");
        }}>목록으로</button
    >
</div>
