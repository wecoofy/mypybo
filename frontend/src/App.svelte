<script>
  import Router from "svelte-spa-router";

  import Home from "./routes/Home.svelte";
  import Detail from "./routes/Detail.svelte";
  import QuestionCreate from "./routes/QuestionCreate.svelte";
  import Nav from "./components/Nav.svelte";
  import UserCreate from "./routes/UserCreate.svelte";
  import UserLogin from "./routes/UserLogin.svelte";
  import QuestionModify from "./routes/QuestionModify.svelte";
  import AnswerModify from "./routes/AnswerModify.svelte";

  const routes = {
    "/": Home,
    "/detail/:q_id": Detail,
    "/question-create": QuestionCreate,
    "/user-create": UserCreate,
    "/user-login": UserLogin,
    "/question-modify/:question_id": QuestionModify,
    "/answer-modify/:answer_id": AnswerModify,
  };
</script>

<Nav />
<Router {routes} />
