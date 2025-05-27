<script>
  import apicall from "../lib/api";
  import { link } from "svelte-spa-router";
  import { page, is_login, keyword } from "../lib/store";
  import moment from "moment/min/moment-with-locales";
  moment.locale("ko");

  let question_list = [];
  let size = 10;
  let total = 0;
  let kw = "";
  $: total_page = Math.ceil(total / size);

  function get_question_list() {
    let params = {
      page: $page,
      size: size,
      keyword: $keyword,
    };
    apicall("get", "/api/question/list", params, (json) => {
      question_list = json.question_list;
      total = json.total;
      kw = $keyword;
    });
  }

  $: $page, $keyword, get_question_list();
</script>

<div class="container my-3">
  <div class="row my-3">
    <div class="col-6">
      <a use:link href="/question-create" class="btn btn-primary {$is_login ? '' : 'disabled'}"> 질문 등록하기 </a>
    </div>
    <div class="col-6">
      <div class="input-group">
        <input type="text" class="form-control" bind:value={kw} />
        <button
          class="btn btn-outline-secondary"
          on:click={() => {
            ($keyword = kw), ($page = 0);
          }}
        >
          찾기
        </button>
      </div>
    </div>
  </div>
  <table class="table">
    <thead>
      <tr class="text-center table-dark">
        <th>ID</th>
        <th style="width: 50%">제목</th>
        <th>글쓴이</th>
        <th>작성일시</th>
      </tr>
    </thead>
    <tbody>
      {#each question_list as question}
        <tr class="text-center">
          <td>{question.id}</td>
          <td class="text-start">
            <a use:link href="/detail/{question.id}">{question.subject}</a>
            {#if question.answers.length > 0}
              <span class="text-danger small mx-2">{question.answers.length} </span>
            {/if}
          </td>
          <td>{question.user ? question.user.username : ""}</td>
          <td>
            {moment(question.create_date).format("YYYY년 MM월 DD일 HH:mm")}
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
  <!-- 페이징처리 시작 -->
  <ul class="pagination justify-content-center">
    <!-- <li class="page-item"><button class="page-link" on:click={() => console.log("page: " + $page + ", total: " + total_page)}>로그</button></li> -->
    <!-- 처음 -->
    <li class="page-item"><button class="page-link" style="width: 6ch;" on:click={() => ($page = 0)}>처음</button></li>
    <!-- 이전페이지 -->
    <li class="page-item {$page <= 0 && 'disabled'}">
      <button class="page-link" style="width: 6ch;" on:click={() => $page--}>이전</button>
    </li>
    <!-- 페이지번호 -->
    {#each Array(total_page) as _, loop_page}
      {#if loop_page >= $page - 5 && loop_page <= $page + 5}
        <li class="page-item {loop_page === $page && 'active'} text-center">
          <button on:click={() => ($page = loop_page)} style="width: 4ch;" class="page-link">{loop_page + 1}</button>
        </li>
      {/if}
    {/each}
    <!-- 다음페이지 -->
    <li class="page-item {$page >= total_page - 1 && 'disabled'}">
      <button class="page-link" style="width: 6ch;" on:click={() => $page++}>다음</button>
    </li>
    <!-- 마지막 -->
    <li class="page-item">
      <button class="page-link" style="width: 6ch;" on:click={() => ($page = total_page - 1)}>끝</button>
    </li>
  </ul>
  <!-- 페이징처리 끝 -->
</div>
